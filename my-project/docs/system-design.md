# System Design — Olympiad Portal

> **Team:** Prompt Engineers
> **Version:** 1.0
> **Last updated:** September 2026

---

## 1. Architecture

The Olympiad Portal uses a **multi-tier architecture** with three tiers deployed on separate managed services:

```
CLIENT TIER                    APPLICATION TIER               DATA TIER
React SPA (Vercel CDN)   --->  Express API (Render)   --->   PostgreSQL (Supabase)
Role-based dashboards          Routes > Controllers          Prisma ORM
React Router, React Query      > Services > Prisma           Supabase Auth
                               Helmet, CORS, WebSocket       Supabase Storage
```

### Why Multi-Tier

- **Separation of concerns:** Each tier handles one responsibility (presentation, business logic, data storage)
- **Independent scaling:** Frontend serves from CDN; backend scales horizontally on Render; database scales vertically on Supabase
- **Security boundaries:** The client never accesses the database directly; all data flows through the API tier which enforces authentication and authorisation

For the rationale behind each individual technology choice, see [Architecture](architecture.md), [Frontend](frontend.md), [Backend](backend.md), and [Data and Storage](data-storage.md).

---

## 2. Backend Architecture

### Request Lifecycle

```
HTTP Request
  > Helmet (security headers)
  > CORS (origin validation)
  > Body parsing (JSON / URL-encoded)
  > Route matching
  > requireAuth middleware (validates Supabase JWT)
  > requireRole middleware (checks role: organiser/educator/student)
  > express-validator middleware (validates request body/params)
  > Controller (HTTP concerns)
  > Service (business logic)
  > Prisma ORM (database queries)
  > JSON Response
  > errorHandler (catches all errors, maps to HTTP status codes)
```

### Error Handling

Custom error hierarchy maps to HTTP status codes:

| Error Class     | Status | Use case                                   |
| --------------- | ------ | ------------------------------------------ |
| BadRequest      | 400    | Invalid input                              |
| Unauthorized    | 401    | Missing or expired JWT                     |
| Forbidden       | 403    | Insufficient role                          |
| NotFound        | 404    | Resource does not exist                    |
| Conflict        | 409    | Unique constraint violation (Prisma P2002) |
| ValidationError | 422    | Failed express-validator rules             |

All errors return `{ success: false, error: "message" }`. Stack traces are suppressed in production.

### API Routes

Routes are grouped by resource: auth/registration, olympiads, rounds, notification rules, questions, exam approval, papers, submissions, marking, and the student portal/exam-sitting flow. Access is restricted by role (public, any authenticated user, organiser, or educator) via the `requireAuth`/`requireRole` middleware described above.

The full, current list of endpoints with request/response shapes lives in one place to avoid drift: see the [API Reference](api.md).

---

## 3. Database Design

> This section documents `backend/prisma/schema.prisma` as it actually exists in the codebase — not an aspirational or planned schema. It is regenerated whenever the schema changes so the two never drift apart.

### Entity-Relationship Diagram

```
users ──┬── invitations (sent_by / claimed_by)
        ├── school_memberships ──── schools ──┬── educators
        │                                      ├── entrants
        │                                      ├── invitations
        │                                      └── school_registrations ── olympiads
        ├── educators
        ├── entrants
        ├── olympiads (organiser) ──┬── rounds ──┬── papers ── questions ── answers
        │                            │            ├── submissions ── answers
        │                            │            ├── entrant_registrations
        │                            │            └── results
        │                            ├── school_registrations
        │                            └── notification_rules
        └── answers (marked_by)
```

### Models

Sixteen models, grouped by domain. "Constraints" lists anything beyond a plain column: primary keys, uniqueness, defaults, and `onDelete` behaviour.

#### Identity & access

| Model | Fields | Constraints | Purpose |
|---|---|---|---|
| `users` | id, email, full_name, auth_provider_id?, created_at, deleted_at?, role | PK `id` (uuid); `email` unique | Core account for every organiser, educator, and student. `deleted_at` supports soft delete so historical submissions/results stay attributable after account deletion. |
| `invitations` | id, code, token_hash?, type, intended_school_role?, school_name?, olympiad_id?, school_id?, email?, used_by_id?, accepted_at?, created_by_id?, expires_at, created_at | PK `id`; `code` unique; `token_hash` unique; indexes on `code`, `school_id`, `olympiad_id` | Backs both invitation mechanisms described in the API reference: short codes (`code`) and emailed link tokens (`token_hash`, stored hashed rather than in plaintext so a leaked log line can't be used to claim an invite). `intended_school_role` lets a single invitation pre-assign `coordinator` vs `educator` before the invitee even exists as a user. |
| `schools` | id, name, address?, created_at | PK `id` | A registered school. |
| `school_memberships` | id, user_id, school_id, role, status, created_at | PK `id`; unique `[user_id, school_id]`; index `[school_id, role, status]` | The permission layer for school-side access: a user's `role` (`coordinator`/`educator`) and `status` (`active`/`removed`) at a given school. This is what the API's "coordinator" checks (e.g. inviting another educator) actually query — see below for why it's separate from `educators`. |
| `educators` | id, user_id, school_id, created_at | PK `id`; `user_id` unique | The stable identity that owns educator-authored records (`submissions.submitted_by`, `entrant_registrations.registered_by`). One row per user, one school. |
| `entrants` | id, user_id?, school_id, full_name, grade?, external_ref?, created_at | PK `id`; `user_id` unique | A student participant. `user_id` is nullable because an entrant can exist (registered by their educator) before they ever sign up for their own login; `external_ref` supports matching entrants against an external school register. |

#### Competition structure

| Model | Fields | Constraints | Purpose |
|---|---|---|---|
| `olympiads` | id, name, organiser_id, timezone, created_at | PK `id`; default `timezone = "Africa/Johannesburg"` | A competition owned by one organiser. |
| `school_registrations` | id, school_id, olympiad_id, registered_at, status | PK `id`; unique `[school_id, olympiad_id]`; default `status = approved` | Many-to-many join between schools and olympiads. |
| `rounds` | id, olympiad_id, name, notes?, opens_at, closes_at, results_release_at?, state, exam_status, approved_by?, approved_at?, qualifying_threshold?, created_at | PK `id`; index `[state, opens_at, closes_at]` | A round within an olympiad. `state` (scheduled → open → closed → marking → results_released) drives the overall timeline; `exam_status` (draft → ready → live) is a *separate* gate specifically for online sitting, so an organiser can prep and approve the exam independently of whether the round has opened yet. The composite index supports the lifecycle poller's query for rounds crossing a time threshold. |
| `notification_rules` | id, olympiad_id, name, trigger, offset_minutes, recipient, channel, condition?, enabled, created_at, updated_at | PK `id`; index `[olympiad_id, enabled]` | Organiser-configured comms rules (e.g. "remind registered educators 60 minutes before a round closes"). |

#### Content & submissions

| Model | Fields | Constraints | Purpose |
|---|---|---|---|
| `papers` | id, round_id, title, file_url?, memo_file_url?, is_archived, released_at?, created_at | PK `id` | The question paper and memo for a round. |
| `questions` | id, paper_id, type, prompt, options?, correct_option?, correct_answer?, negative_pct, max_points, created_at | PK `id`; defaults `negative_pct = 0`, `max_points = 1` | One question on a paper (`mcq`, `true_false`, `short_answer`, or `essay`). |
| `submissions` | id, round_id, entrant_id, submitted_by?, route, status, is_late, idempotency_key, received_at, round_close_snapshot, created_at | PK `id`; `idempotency_key` unique; unique `[round_id, entrant_id, route]` | One entrant's attempt at a round via one route. `idempotency_key` guarantees a retried network request is never double-counted. `round_close_snapshot` freezes the round's `closes_at` value *at the moment the submission is received*, so a later edit to the round's schedule can't retroactively change whether a past submission was on time. |
| `answers` | id, submission_id, question_id, answer_value?, is_correct?, points_awarded?, marked_by?, marked_at? | PK `id`; unique `[submission_id, question_id]` | One answer to one question within a submission; carries its own marking state and marker. |
| `entrant_registrations` | id, entrant_id, round_id, registered_by, status, created_at | PK `id`; unique `[entrant_id, round_id]`; default `status = "registered"` | Which entrants are entered for which round, and by which educator. |
| `results` | id, round_id, entrant_id, total_points, rank?, qualified, finalised_at? | PK `id`; unique `[round_id, entrant_id]`; index `[round_id, rank]` | The finalised score/rank/qualification for one entrant in one round, written once marking is complete rather than computed on every read — standings are read far more often than they're recalculated. |

### Enums

| Enum | Values |
|---|---|
| `user_role` | `organiser`, `educator`, `student` |
| `invite_type` | `school`, `educator`, `student` |
| `school_role` | `coordinator`, `educator` |
| `membership_status` | `active`, `removed` |
| `registration_status` | `pending`, `approved`, `withdrawn` |
| `round_state` | `scheduled`, `open`, `closed`, `marking`, `results_released` |
| `exam_status` | `draft`, `ready`, `live` |
| `submission_route` | `online`, `offline` |
| `submission_status` | `received`, `automarked`, `queued_for_marking`, `marked`, `moderated` |
| `notification_trigger` | `round_opens`, `round_closes`, `results_released` |
| `notification_recipient` | `registered_educators`, `registered_schools`, `qualifying_entrants` |
| `notification_channel` | `email` |

### Design Decisions

- **UUIDs (`gen_random_uuid()`) for all primary keys** — generated in Postgres, safe to expose in URLs, and don't leak insertion order or row counts the way a sequential ID would.
- **Soft delete only on `users`** (`deleted_at`) — a deleted account's historical submissions, results, and marking records stay intact and attributable rather than cascading into orphaned or deleted rows.
- **`timestamptz(6)` on every temporal column** — timezone-aware precision, needed both for deadline enforcement today and for the multi-timezone, multi-olympiad case the schema already has room for (`olympiads.timezone`).
- **`school_memberships` alongside `educators`, not instead of it** — `educators` is the stable identity that other tables' foreign keys point to (`submissions.submitted_by`, `entrant_registrations.registered_by`), so it isn't churned as permissions change. `school_memberships` is the more flexible role/status layer actually queried for authorisation (e.g. "does this user have an active `coordinator` membership at this school?"), and its `status` field lets access be revoked (`removed`) without deleting the historical link.
- **`invitations.token_hash` stored hashed, separately from `code`** — link-based invites and short-code invites are both supported, but a hashed, independently-unique token means a link token can't be brute-forced or replayed the way a short human-typed code more plausibly could be.
- **Composite unique constraints** — `submissions(round_id, entrant_id, route)` stops the same entrant submitting twice via the same route; `entrant_registrations(entrant_id, round_id)` stops double registration; `answers(submission_id, question_id)` stops duplicate answer rows.
- **`submissions.round_close_snapshot`** — captures the round's close time at the moment of receipt, so a dispute over "was this on time" is answered from an immutable record even if the round's schedule is edited afterwards.
- **`submissions.idempotency_key`** — enforced as a database-level unique constraint (not just application logic), so a retried request from a flaky connection can never be counted twice.
- **`rounds.exam_status` kept separate from `rounds.state`** — lets an organiser prepare and approve the online exam (`draft` → `ready` → `live`) independently of the round's own open/closed timeline.
- **Indexes chosen for known query patterns** — `rounds(state, opens_at, closes_at)` for the lifecycle service's polling query; `results(round_id, rank)` for standings/leaderboard reads.

### Deployment

The schema is deployed as managed **PostgreSQL on Supabase** (see [Data and Storage](data-storage.md) for why Supabase was chosen). `backend/prisma/schema.prisma`, shown above, is the single source of truth: schema changes are made there, Prisma generates a migration, and the same `DATABASE_URL` environment variable is used to apply it in local development and against the hosted Supabase instance in deployment (see [CI/CD and Hosting](cicd-hosting.md) for the deployment pipeline itself). The generated Prisma client (`../generated/prisma`) gives the backend type-safe queries against this exact schema, so a mismatch between the documented schema and the running database would fail at build time rather than surface as a runtime bug.

---

## 4. Core Behaviours

### Authentication Flow

1. **Organiser self-registration:** POST /auth/register/organiser with organiser_secret environment variable, creates Supabase Auth user and users row with role=organiser
2. **School invitation:** Organiser creates school, generates SCH-XXXX-XXXX code, emails school contact
3. **Educator registration:** Educator visits /signup, enters code, verifies school details, creates account linked to school
4. **Student registration:** Educator bulk-generates codes, each maps to an entrant record, student signs up with code

All API requests include `Authorization: Bearer <access_token>`. The requireAuth middleware validates the JWT via Supabase. The requireRole middleware checks user.role.

### Round Lifecycle State Machine

```
scheduled  >  open  >  closed  >  marking  >  results_released
```

- **scheduled:** Round created, opens_at is in the future. No access to papers.
- **open:** opens_at reached. Papers downloadable, submissions accepted, online sitting available.
- **closed:** closes_at reached. No more submissions. Papers locked.
- **marking:** Organiser triggers auto-mark. MCQ scored automatically; non-MCQ queued for human marking.
- **results_released:** Scores visible. Papers auto-archived. Qualified entrants identified.

Transitions from scheduled to open and open to closed are automatic, driven by a lifecycle service that queries rounds where the current time has passed the configured threshold.

### Auto-Marking Pipeline

1. Fetch all submissions with status=received for the round
2. For each submission, iterate over answers
3. MCQ answers: compare answer_value to correct_option, award max_points or 0
4. Non-MCQ answers: set submission status to queued_for_marking for human review
5. Update submission status to automarked or queued_for_marking

### Submission Idempotency

The composite unique constraint on (round_id, entrant_id, route) plus a unique idempotency_key ensures each submission is counted exactly once, even if the request is retried due to network issues.

---

## 5. Security

| Area             | Implementation                                                     |
| ---------------- | ------------------------------------------------------------------ |
| Passwords        | Min 8 chars, uppercase letter, number required                     |
| JWT              | Managed by Supabase Auth; access tokens expire after 1 hour        |
| HTTP headers     | Helmet sets CSP, HSTS, X-Content-Type-Options, X-Frame-Options     |
| CORS             | Restricted to frontend domain (Vercel URL in production)           |
| Input validation | express-validator on all routes with sanitisation                  |
| SQL injection    | Prevented by Prisma parameterised queries                          |
| File upload      | Multer with size limits; PDF only for papers/memos                 |
| Enumeration      | UUIDs prevent sequential ID guessing                               |
| Access control   | requireRole on every protected route                               |
| Time-gating      | Papers locked before opens_at; submissions blocked after closes_at |
| Results embargo  | Scores only visible when round state is results_released           |

---

## 6. Backlog Feature Design (Not Yet Implemented)

### Online Paper Sitting (Sprint 2)

Students sit papers in the browser with a server-authoritative timer. The remaining time is calculated from round.closes_at minus submission.received_at on the server, not a client-side countdown. This prevents timer manipulation via page refresh.

Answers are auto-saved on each change via debounced PATCH requests. On reconnection after a dropped connection, the frontend fetches the submission, pre-fills saved answers, and resumes the server-calculated timer.

The interface includes a question navigator showing answered, unanswered, and flagged questions, with Previous/Next navigation and a flag-for-review toggle.

### Notification System (Sprint 2)

Organisers configure notification rules with three components: trigger (e.g., "round opens in 24h"), recipient (e.g., "all registered educators"), and channel (email). A scheduler checks upcoming events against configured rules and dispatches notifications.

Rules can be dry-run tested against a past round before activation.

### Human Marking Queue (Sprint 2-3)

Non-MCQ answers are queued with status=queued_for_marking. Markers see a queue sorted by round and question, award partial credit (0 to max_points), and progress through the queue. Moderators sample a percentage of each marker's work for consistency review.

### Progression and Standings (Sprint 2-3)

After results are generated, entrants whose total_points meet the qualifying_threshold are automatically registered for the next round (no manual re-registration). Standings are published at both individual and school level.

### Question Bank and Variants (Sprint 3)

A central question repository tagged by subject, difficulty, topic, and type. Papers can be generated from the bank with balanced difficulty. Variants give different entrants different question orders or equivalent questions, and marking uses each entrant's specific variant key.

Question difficulty is derived from actual performance data: difficulty = 1 - (average score / max points).

### Appeals Pipeline (Sprint 3)

Students request a remark, which creates an appeal assigned to a reviewer. If upheld, marks are adjusted, results recalculated, ranks recomputed, and qualification status rechecked automatically.

### Multi-Olympiad Support (Sprint 3)

A single organiser account manages multiple concurrent olympiads, each with its own timezone setting. The authoritative clock for deadline enforcement uses the olympiad's timezone. All olympiads share a single archive of past papers.

### Online/Offline Hybrid (Sprint 3)

The same paper is available both online (browser sitting) and offline (downloaded PDF administered on paper). Offline results are uploaded by the educator. Both routes merge into a single result set per entrant. If an entrant appears via both routes, the system flags it for organiser review.

### Audit Log and Replay (Sprint 3)

Every automatic action (round transitions, auto-marking, progression, notifications) is logged with timestamp, action type, actor, and details. Given a round_id, the full event history can be replayed for dispute investigation.

---

## 7. Deployment Architecture

```
Internet
  |
  +---> Vercel CDN ---> React SPA (static assets, auto-deploy from GitHub)
  |
  +---> Render    ---> Express API (Node.js, auto-deploy from GitHub)
              |
              +---> Supabase ---> PostgreSQL + Auth + Storage
```

Environment variables are managed per environment:

- Backend: DATABASE_URL, SUPABASE_URL, SUPABASE_SERVICE_KEY, ORGANISER_SECRET, FRONTEND_URL
- Frontend: VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY

For why these specific hosting providers were chosen, see [CI/CD and Hosting](cicd-hosting.md).

---