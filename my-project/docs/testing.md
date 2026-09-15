# Olympiad Portal — Testing Documentation

This document covers how the Olympiad Portal is tested:

1. [Testing approach](#1-testing-approach)
2. [Automated testing procedure](#2-automated-testing-procedure): every type of automated test, why it is used, what it covers and how to run it
3. [Test policy](#3-test-policy): the rules the team follows for writing and running tests
4. [User feedback process](#4-user-feedback-process): how feedback from users is collected and acted on

Bugs found during testing and user feedback are recorded in [bug-tracking.md](bug-tracking.md).

---

## 1. Testing approach

The portal has a React frontend and an Express + PostgreSQL backend used by three roles: organisers, educators and students. Tests are written at several levels. Each level answers a different question, from "does this function work?" to "can a real user finish their task in a real browser?".

| Level | Question it answers | Speed | Status |
|---|---|---|---|
| Unit tests | Does each function, component and hook work on its own? | Very fast | Done |
| Integration tests | Do the parts work together (services with a real database, full pages with the app's real logic)? | Fast | Done |
| API tests | Does every endpoint return the right status, data and errors, and enforce who may call it? | Fast | Done |
| End-to-end (E2E) tests | Can each role complete their real tasks in a real browser? | Slower | In progress |
| Continuous integration (CI) | Do all of the above pass on every push, before code is merged or deployed? | Automatic | Planned |

Most tests are unit, integration and API tests, because they are fast and point to the exact cause of a failure. A smaller number of E2E tests cover the most important user journeys from start to finish.

### Current results

| Suite | Location | Tests |
|---|---|---|
| Backend unit | `backend/tests/unit` | 676 |
| Backend integration | `backend/tests/integration` | 111 |
| Backend API | `backend/tests/api` | 475 |
| Frontend unit | `frontend/tests/unit` | 247 |
| Frontend integration | `frontend/tests/integration` | 123 |
| **Total** | | **1,632** |

| Coverage | Statements | Branches | Functions | Lines |
|---|---|---|---|---|
| Backend unit | 99.4% | 97.5% | 99.6% | 99.5% |
| Backend integration | 86.8% | 71.1% | 95.2% | 89.7% |
| Backend API | 94.6% | 82.5% | 97.4% | 95.5% |
| Frontend unit | 99.2% | 98.6% | 99.3% | 99.1% |

---

## 2. Automated testing procedure

### 2.1 Tools and why they were chosen

| Tool | Used for | Why |
|---|---|---|
| **Jest** | Backend unit, integration and API tests | The standard test runner for Node.js. It includes mocking, coverage reports and parallel runs, so no extra libraries are needed. |
| **Vitest** | Frontend unit and integration tests | Built for Vite, which the frontend already uses, so tests share the app's build configuration. Its API is almost the same as Jest's, so one style is used across the project. |
| **React Testing Library** + **user-event** | Frontend tests | Tests use the page the way a user does: finding buttons by their label, typing and clicking. They check what the user sees, not internal code, so they keep working when code is reorganised. |
| **jsdom** | Frontend tests | Simulates a browser inside Node, so components render without opening a real browser. |
| **MSW** (Mock Service Worker) | Frontend integration tests | Intercepts network requests and returns realistic backend responses. Pages run their real API client and logic, but no real server is needed. |
| **PGlite** | Backend integration and API tests | The real PostgreSQL engine running inside the test process. Tests use a genuine Postgres database built from the project's migrations, with nothing to install (no Docker, no local database, no cloud database). |
| **Supertest** | Backend API tests | Sends real HTTP requests to the Express app without starting a server on a port, and checks status codes, headers and response bodies. |
| **Playwright** | E2E tests | Drives real browsers (Chromium, Firefox and WebKit) with one install. It runs headless in CI with no separate GUI runner, and records screenshots, videos and traces when a test fails. |

### 2.2 Folder layout

```
backend/
  jest.config.js                 unit tests
  jest.integration.config.js     integration tests
  jest.api.config.js             API tests
  tests/
    unit/                        controllers, services, middleware, routes, jobs, utils
      helpers/                   database mock, fake request/response, controller test cases
    integration/                 services and database working together
      setup/                     starts the test database and empties it before each test
      helpers/                   test data factories, fake Supabase and email
    api/                         HTTP tests for every endpoint
      helpers/                   request helpers and the list of all endpoints

frontend/
  vitest.config.ts               "unit" and "integration" projects
  tests/
    unit/                        components, hooks, helpers
    integration/                 full pages through the real app
      support/                   app renderer, network mocks, API-shaped test data

e2e/                             Playwright E2E tests (in progress)
```

### 2.3 Unit tests

**What they are:** tests of a single function, component or hook, with everything it depends on replaced by a mock.

**Why they are used:** they run in seconds, so developers can run them after every change. A failing unit test points to one exact piece of code, which makes problems quick to find. They also make it practical to check every branch of the logic, such as each validation rule and each error case.

**Backend: what is tested**
- **Services:** the business logic, meaning authentication and invitations, olympiads, rounds, questions, papers, submissions, marking, marking maths, results, the round lifecycle, schools, notification rules, and the educator and student views.
- **Controllers:** each one calls the right service with values from the request and returns the right status code and response.
- **Middleware:** authentication and roles, validation, error handling and rate limiting.
- **Routes with their own logic:** exam sitting and exam status.
- **Other:** the scheduled round-lifecycle job, the email sender, invitation code helpers and error classes.

**Backend: how**
- The database client (Prisma) is replaced by a mock, so each test decides what the database returns and checks what the code asked for.
- Supabase and email are mocked, so no external service is contacted.

**Frontend: what is tested**
- UI components: buttons, form controls, display components and the page layout.
- The API client, Supabase client, login and logout helpers, and data-fetching hooks.
- Page building blocks:
  - the question form and question cards
  - the answer-marking card and the answer-key correction editor
  - the school invitation form
  - round status labels and dashboard helpers
  - exam timer and exam answer storage

**Frontend: how** — each piece is rendered on its own, and its API calls and services are mocked.

### 2.4 Integration tests

**What they are:** tests that check several real parts working together.

**Why they are used:** unit tests mock the pieces around the code, so they can't catch problems in how pieces fit together. Examples are a database constraint, a transaction, a page that reads a field the API doesn't send, or a value arriving as text instead of a number. Integration tests catch these while staying fast and not needing any external service.

#### Backend integration tests

Real services and the real Prisma client work with a real PostgreSQL database.

**Test database strategy**
1. When the test run starts, an in-memory PostgreSQL database (PGlite) is created. Every migration in `prisma/migrations` is applied in order, which also confirms the migrations build a working database from scratch.
2. The app connects to it through its normal database code; only the connection address changes.
3. Every setting the app reads is set to a safe test value before the app loads, so a developer's real `.env` file (with production credentials) is never used.
4. Every table is emptied before each test, so tests never depend on each other. The reset refuses to run against anything except the local test database.
5. Test files run one at a time because they share the database.
6. Test data is created with factories (organisers, olympiads, schools, educators, rounds, papers, questions, learners, students, invitations) so every test starts from a clear, realistic setup.
7. Supabase Auth, Supabase Storage and email are replaced by fakes.

| Test file | What it covers |
|---|---|
| `database.test.js` | All tables are created by the migrations; defaults, decimal values, unique constraints, cascading deletes and delete restrictions |
| `authAndInvitations.test.js` | The full invitation chain (organiser → school → educator → student), rollback when registration fails, authentication and school membership checks |
| `olympiadsAndRounds.test.js` | Creating and managing olympiads and rounds, round state changes, deleting rounds |
| `questionsAndExamStatus.test.js` | Building an exam and the draft → ready → live approval flow |
| `examSitting.test.js` | A student sitting an exam online: load, start, save answers, resume, submit and automatic marking |
| `submissionsMarkingResults.test.js` | Offline submissions, automatic marking, the marking queue, manual marking, generating results and the automatic round lifecycle |
| `educatorAndStudentViews.test.js` | Learner registration with round progression, results, past papers, and the student dashboard and results |
| `schoolsRulesAndPapers.test.js` | School lists, notification rules, paper upload, download and archive |

#### Frontend integration tests

Whole pages run inside the real app, with its real routes, API client, Supabase client and data fetching. Only the network is faked, using MSW.

**How**
- The real application is rendered at a chosen URL, and the test clicks, types and checks results as a user would.
- MSW answers every request with data in exactly the backend's format, and records each request so tests can check what was sent. Any request without a prepared response fails the test.
- Test data matches real API responses, including decimal values sent as text.

| Test file | What it covers |
|---|---|
| `auth.test.tsx` | Log in for each role, organiser sign-up, sign-up with an invitation code or emailed link, error messages, log out |
| `organiserManagement.test.tsx` | Organiser dashboard, olympiads, schools, creating a round and uploading its paper |
| `notificationsAndArchive.test.tsx` | Announcement rules (create, enable, disable, delete) and the paper archive |
| `examReviewAndMarking.test.tsx` | Reviewing an exam against the memo, approving or sending it back, correcting answers, the marking queue and generating results |
| `examBuilder.test.tsx` | Adding each question type, editing, deleting, points totals and submitting the exam for review |
| `educatorPages.test.tsx` | Educator dashboard, round page (paper download, submitting results for each learner), entrants and invitations, results |
| `studentPages.test.tsx` | Student dashboard, round page (paper download, sitting online, results), results page |
| `examSitting.test.tsx` | Timer based on the server clock, autosave, flags, resuming, saving while offline, submitting when the connection returns, automatic submit when time runs out |

### 2.5 API tests

**What they are:** real HTTP requests sent to the backend, going through every layer (routing, security middleware, validation, business logic and the database) and checking the actual response.

**Why they are used:** the frontend relies on the API behaving exactly as documented. API tests prove every endpoint returns the right status code, data and error message, and, most importantly, that each endpoint only lets the right people in. These are the rules that protect exam papers, answer keys, marks and learner data.

**How**
- Supertest sends requests to the Express app.
- The database is a real PostgreSQL database, set up exactly as in the integration tests.
- Supabase Auth is faked so a test can sign in as any user, while user accounts and roles are still loaded from the real database.
- Every response is checked against the standard format: `{ success, data }` or `{ success, error }`.

**Every endpoint is covered.** All 66 endpoints are listed in `backend/tests/api/helpers/endpoints.js` with the roles allowed to call each one. Using that list, the tests automatically check that:
1. The list matches the route files exactly. If a new endpoint is added without being listed, the test run fails.
2. Every protected endpoint returns **401** when no login token is sent.
3. Every protected endpoint returns **403** for each role that is not allowed.
4. Public endpoints can be reached without logging in.

For each area, the tests then cover:
- **The normal case:** correct request, correct response and data saved.
- **Ownership:** users from another olympiad or school are refused.
- **Validation:** invalid input returns **422** with a clear message.
- **Business rules and edge cases:** for example, a round that isn't open yet, a duplicate submission or a file over the size limit.

| Test file | What it covers |
|---|---|
| `endpoints.test.js` | The endpoint list, and authentication and role checks for every endpoint |
| `app.test.js` | Health check, unknown routes, security headers, CORS, rate limiting, malformed requests and invalid IDs, login token checks |
| `auth.test.js` | Registration, invitation codes and links, the current user, account deletion, every invitation endpoint |
| `olympiadsAndRounds.test.js` | Olympiads, rounds, schools, archive, notification rules, editing, state changes, deleting and paper upload |
| `examContent.test.js` | Questions, the exam approval flow, and links to papers and memos |
| `submissionsAndMarking.test.js` | Offline submissions, automatic marking, the marking queue, marking answers and generating results |
| `educator.test.js` | All educator portal endpoints |
| `student.test.js` | Student dashboard, round details, results, paper download and the full online exam over HTTP |

### 2.6 End-to-end (E2E) tests — in progress

**What they are:** tests that open the real portal in a real browser and complete tasks exactly as users would, with the real frontend, real backend and a real database working together.

**Why they are used:** they are the final check that the whole system works for its users. They catch problems that only appear when everything runs together: real login, file uploads, browser behaviour, page navigation and layout. Only the most important journeys are covered, because E2E tests are slower than the other levels.

**Why Playwright**
- **One install covers three browsers:** Chromium, Firefox and WebKit (Safari's engine).
- **Runs in CI without a separate GUI runner:** it works headless, which suits the project's Gitea Actions pipeline.
- **Built-in failure evidence:** screenshots, videos and step-by-step traces.
- **Waits automatically** for pages and elements, which keeps tests reliable.

**Test environment**
- Playwright starts the frontend and backend itself before the tests run.
- The backend uses a dedicated test database, filled with known starting data (seeded) before the run, and a separate Supabase test project. Production data and accounts are never used.
- Each test signs in with its own test account for its role.

**Planned user journeys**

| Journey | Role(s) |
|---|---|
| Log in and reach the correct dashboard; log out | All roles |
| Create an olympiad and a round, upload the question paper and memo | Organiser |
| Invite a school, then register as its coordinator from the invitation | Organiser, educator |
| Register learners and invite students | Educator |
| Build the exam from the question paper and submit it for review | Educator |
| Review the exam against the memo and approve it | Organiser |
| Sit the exam online: answer, flag, resume after a refresh, submit | Student |
| Submit offline results for learners | Educator |
| Mark essay answers and release results | Organiser |
| View results and download the paper | Student, educator |
| Pages remain usable at mobile width | All roles |

**How to run (once added)**
```bash
cd e2e
npm install
npx playwright install      # downloads the browsers
npx playwright test         # runs all E2E tests headless
npx playwright show-report  # opens the HTML report with screenshots and traces
```

### 2.7 How to run the tests

No database, `.env` file or internet connection is needed for the unit, integration or API tests.

**Backend**
```bash
cd backend
npm install
npm run test:unit                   # unit tests
npm run test:integration            # integration tests
npm run test:api                    # API tests
npm test                            # all backend tests
npm run test:coverage               # unit test coverage report
npm run test:integration:coverage   # integration test coverage report
npm run test:api:coverage           # API test coverage report
```

**Frontend**
```bash
cd frontend
npm install
npm run test:unit          # unit tests
npm run test:integration   # integration tests
npm test                   # all frontend tests
npm run test:watch         # re-runs unit tests on every save
npm run test:coverage      # unit test coverage report
npm run typecheck:tests    # type-checks the test code
```

### 2.8 Continuous integration — planned

The project already uses Gitea Actions to build and deploy the backend (Render) and the frontend (Vercel). The test suites are being added to that pipeline:

1. On every push and pull request, the pipeline installs dependencies, then runs linting, a frontend build, and the backend and frontend test suites.
2. E2E tests run on pull requests into `main`.
3. Deployment to Render and Vercel happens only when every test passes.
4. Coverage reports and Playwright reports are saved with each pipeline run.

---

## 3. Test policy

These rules apply to everyone working on the project.

### 3.1 When tests must be written
1. **New features** include tests at the levels that fit:
   - unit tests for new logic
   - an integration test for new database behaviour or a new page
   - API tests for every new or changed endpoint
2. **New endpoints** must be added to `backend/tests/api/helpers/endpoints.js` with the roles allowed to call them; the test run fails until this is done.
3. **Every bug fix includes a test** that fails before the fix and passes after it, so the bug cannot return. The bug and its fix are recorded in [bug-tracking.md](bug-tracking.md).
4. **Changes to permissions or security** (who may see or change data) always include API tests for both the allowed and the refused users.

### 3.2 Before code is merged
1. All test suites pass, along with linting and the frontend build.
2. No test is skipped, deleted or weakened just to make a change pass. If a test's expected behaviour changes, the reason is explained in the pull request.
3. Coverage must not drop noticeably. Target: at least 90% of statements for unit tests.
4. Pull requests are reviewed by another team member, including the tests.

### 3.3 How tests are written
1. **Tests are independent.** Each test creates its own data and never depends on another test having run first. The test database is emptied before every test.
2. **No real external services in automated tests.** Supabase and email are faked in unit, integration and API tests, and tests never use production data or credentials. E2E tests use separate test accounts and a test database only.
3. **Test behaviour, not internal code.** Frontend tests find elements by role, label or visible text, as a user would. Backend tests check responses and saved data.
4. **Realistic data.** Mocked API responses use exactly the backend's format, and test records are built with the shared factories.
5. **Clear names.** Test names describe the expected behaviour, for example "responds 403 to another organiser".
6. **Test code is held to the same standard as app code.** It is readable, reviewed and type-checked on the frontend.

### 3.4 Handling test failures
1. A failing test on `main` is fixed before any other work is merged.
2. A test that fails only sometimes (a "flaky" test) is investigated and fixed rather than re-run until it passes.
3. Failures found during testing are logged in [bug-tracking.md](bug-tracking.md).

---

## 4. User feedback process

Feedback from real users (organisers, educators and students) is collected in a formal way, so it is recorded, prioritised and followed through to a fix.

### 4.1 Collecting feedback
1. **User testing sessions.** Before each release, representative users from each role are given set tasks. Examples: "create a round and upload its paper", "register three learners", "sit the exam and submit". Observers note where users struggle, make mistakes or ask for help.
2. **Feedback form.** After each session, participants complete a short form covering:
   - how easy each task was (1–5)
   - what was confusing
   - what went wrong
   - what they would change
3. **Ongoing reports.** Users can report problems or suggestions to the team at any time. Each report includes the role, the page, what they did, what they expected and what happened, with a screenshot where possible.

### 4.2 Recording and triage
1. Every piece of feedback is logged as an issue in the project's Gitea repository, labelled `bug`, `usability` or `feature request`.
2. Each issue is given a priority:

| Priority | Meaning | Example |
|---|---|---|
| Critical | Data is lost or exposed, or users cannot complete a core task | Exam answers not saved; a user can see or change data they shouldn't |
| High | A core task works but gives wrong or confusing results | Wrong totals; an action appears to fail when it succeeded |
| Medium | Inconvenient but has a workaround | A field is awkward to edit |
| Low | Cosmetic or minor | Wording, spacing, pluralisation |

3. Issues are reviewed at the team's weekly meeting, assigned to a team member and scheduled. Critical issues are handled immediately.

### 4.3 Acting on feedback
1. The reported problem is reproduced. Where possible, an automated test is written that shows the problem.
2. The fix is made, and the new test and all existing tests must pass (see the [test policy](#3-test-policy)).
3. The fix is recorded in [bug-tracking.md](bug-tracking.md) and the issue is closed with a link to the change.
4. Where the feedback came from a user, the fix is confirmed with that user or in the next user testing session.
5. Recurring themes (for example, several users confused by the same page) are raised at the weekly meeting and turned into design improvements.
