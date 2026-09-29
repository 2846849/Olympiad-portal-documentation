# Project Roadmap — Olympiad Portal

> **Team:** Prompt Engineers
> **Version:** 1.0
> **Last updated:** September 2026

---

## Overview

This roadmap maps all user stories across four sprints. Progress is tracked on Trello. Sprint statuses: Done, In Progress, Planned, Blocked.

For the same stories grouped by who they're for instead of by sprint, see [User Stories by Role](user-stories-by-role.md).

```
Week  1    2    3    4    5    6    7    8    9   10   11   12
      |-----------||-----------||-----------||-----------|
      |  SPRINT 1 ||  SPRINT 2 ||  SPRINT 3 ||  SPRINT 4 |
      | B-tier +  || I-tier +  || A-tier +  || Polish +  |
      | Auth      || Online    || Advanced  || Testing   |
      |           || Sitting   || Features  ||           |
```

---

## Sprint 1 — Foundation (B-tier + Authentication)

**Goal:** Working MVP with authentication, core CRUD, round lifecycle, and dashboard shells.

### Stories

| ID  | Story                                            | Status      |
| --- | ------------------------------------------------ | ----------- |
| P1  | Sign up, sign in, reset password, delete account | Done |
| B1  | Create olympiad and define rounds                | Done |
| B2  | Register schools against an olympiad             | Done |
| B3  | Register educators for a school                  | Done |
| B4  | Register entrants for a round                    | Done |
| B5  | Set opening/closing time for a round             | Done        |
| B6  | Auto-transition round states                     | Done |
| B7  | Prevent paper download before round opens        | Done |
| B8  | Prevent result submission after round closes     | Done |
| B9  | Upload round paper and memo                      | Done |
| B10 | Auto-archive past papers on release              | Done     |
| B11 | Download current and archived papers             | Done |
| B12 | Upload school results within open window         | Done |
| B13 | Get submission receipt confirmation              | Done     |
| B14 | Auto-mark MCQ submissions                        | Done |
| B15 | Release entrant scores after round closes        | Done     |
| B16 | Organiser console with live round state          | Done |
| B17 | Single place for papers, results, submissions    | Done |

### Deliverables

- Authentication (organiser self-registration, invitation-based registration for educators/students)
- CRUD for olympiads, rounds, schools
- Paper upload/download with time-gating
- MCQ auto-marking
- Round lifecycle state machine
- Dashboard shells
- Deployed to Render + Vercel + Supabase

### Dependencies

- Supabase project: Done
- Express API skeleton: Done
- Auth middleware: Done
- Prisma schema: Done

---

## Sprint 2 — Core Features (I-tier + Online Sitting)

**Goal:** Students sit papers online with timer and auto-save. Full dashboards. Notification system. Unit tests and CI/CD.

### Stories

| ID  | Story                                        | Status  |
| --- | -------------------------------------------- | ------- |
| I1  | Sit a paper online                           | Done    |
| I2  | Visible timer during sitting                 | Done    |
| I3  | Auto-save answers as given                   | Done    |
| I4  | Resume sitting after connection loss         | Done    |
| I5  | Queue non-MCQ for human marking              | Done    |
| I8  | Request a remark                             | Done    |
| I9  | Set qualifying threshold                     | Done    |
| I10 | Auto-carry qualifying entrants to next round | Done    |
| I11 | Remind before round opens/closes             | Done    |
| I12 | Follow up with non-submitting schools        | Done    |
| I13 | Notify when results released                 | Done    |
| I14 | Configure notification rules                 | Done    |
| I15 | Test rules before go-live                    | Done    |
| I16 | Standings per school and per entrant         | Done    |
| I18 | Student login for papers and results         | Done    |
| I19 | Educator dashboard of entrants and results   | Done    |

### Deliverables

- Online paper sitting interface (timer, navigator, auto-save, flag)
- Server-authoritative timer
- Reconnection support
- Human marking queue
- Remark request workflow
- Automatic progression
- Notification engine
- Standings at school and entrant level
- Unit test suite
- CI/CD pipeline (Gitea Actions)

---

## Sprint 3 — Advanced Features (A-tier)

**Goal:** Full product capability — question bank, variants, multi-olympiad, hybrid submissions, appeals, audit log.

### Stories

| ID  | Story                                       | Status               |
| --- | ------------------------------------------- | -------------------- |
| A1  | Handle round open/close under peak load     | In progress              |
| A2  | Authoritative clock per olympiad timezone   | Done              |
| A3  | Count submission exactly once (idempotency) | Done (schema) |
| A4  | Consistent deadline boundary handling       | Done              |
| A5  | Papers from question bank                   | Done                 |
| A6  | Different variants per entrant              | Done              |
| A7  | Mark against entrant's variant              | Done              |
| A8  | Difficulty from performance data            | Done              |
| A9  | Paper sittable online or offline            | Done                 |
| A10 | Merge online and offline results            | Done              |
| A11 | Reconcile dual-route entrants               | Done              |
| A12 | Appeal a mark                               | Done              |
| A13 | Route appeals to reviewer                   | Done              |
| A14 | Appeal outcome updates standings            | Done              |
| A15 | Multiple concurrent olympiads               | Done                 |
| A16 | Shared paper archive                        | In progress                 |
| A17 | Log every automatic action                  | Done              |
| A18 | Replay round event history                  | Done              |
| I6  | Marker awards partial credit                | In progress              |
| I7  | Moderator reviews marking consistency       | Done              |

### Deliverables

- Question bank with tagging and difficulty calibration
- Paper generation and variant system
- Variant-aware marking
- Appeals pipeline with automatic standings update
- Partial credit and moderation
- Multi-olympiad with per-olympiad timezone
- Shared archive
- Online/offline hybrid with result merging
- Audit log and round replay
- Integration and component tests

---

## Sprint 4 — Polish and QA

**Goal:** Production-ready system. All bugs fixed. E2E tests. Documentation complete.

### Stories

| ID  | Story                                | Status      |
| --- | ------------------------------------ | ----------- |
| P2  | Responsive, accessible on any device | Done        |
| P3  | Hand-written, documented API         | Done        |
| P4  | External service integration         | Done        |
| P5  | Public documentation site            | Done        |

### Deliverables

- Bug bash sessions across all roles
- Accessibility audit (WCAG AA compliance)
- Load testing for concurrent round events
- End-to-end tests for critical flows
- Complete API documentation
- Updated README with setup and deployment guide
- Deployment hardening and environment documentation

---

## Change Log

| Date           | Change                    |
| -------------- | --------------------------|
| September 2026 | Initial roadmap created   |
| September 2026 | Updated completed features|
| September 2026 | Disregarded the email invitation to an Olympiad System and went for an In-Platform invitation System|

This is a living document, reviewed at every Friday retrospective.