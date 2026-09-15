# Comprehensive Development Plan — Olympiad Portal

> **Team:** Prompt Engineers
> **Version:** 1.0
> **Last updated:** September 2026

---

## 1. Project Overview

The Olympiad Portal is a web platform that digitises academic olympiad competitions between schools. It replaces paper-based administration with role-based dashboards for organisers, educators, and students.

**Problem:** Olympiad administration relies on printing papers, manual distribution, physical answer collection, hand-marking, and compiling results in spreadsheets. This is slow, error-prone, and difficult to scale.

**Scope:** Three role-based dashboards covering olympiad creation, school registration, paper distribution, result submission, automated marking, and results release.

**Out of scope:** Payment processing, video proctoring, native mobile apps, OMR scanning, multi-language support.

---

## 2. Team Structure

| #   | Name                 | Primary Role        | Additional Responsibilities                      |
| --- | -------------------- | ------------------- | ------------------------------------------------ |
| 1   | Mashudu Mavhila      | Student Dashboard   | MVP architecture, CI/CD pipeline, DevOps         |
| 2   | Nhlakanipho Mavefua  | Student Dashboard   | Work tracker (Trello), sprint planning           |
| 3   | Madimabe Manenzhe    | Educator Dashboard  | Authentication system, API routes                |
| 4   | Engedzani Mutambedzo | Educator Dashboard  | Documentation lead                               |
| 5   | Kedibone Mnisi       | Organiser Dashboard | Prisma schema, database migrations, data seeding |
| 6   | Andile Dudu          | Organiser Dashboard | UML diagrams, visual design                      |

### Communication

- **Monday:** Sprint planning and task allocation
- **Wednesday:** Mid-week progress and blockers
- **Friday:** Demo completed work, retrospective, update Trello
- **Trello:** Continuous task tracking (Backlog, To Do, In Progress, Review, Done)
- **WhatsApp:** Ad-hoc communication
- **Gitea PRs:** Code review before merging

---

## 3. Methodology

The group follows an Agile, Scrum-based methodology across four sprints, with a fixed weekly communication cadence and Trello as the product backlog. See [Project Methodology](methodology.md) for the full rationale, ceremonies, and sprint structure.

---

## 4. Sprint Detail

Sprint-by-sprint stories, statuses, and deliverables (all four sprints, kept up to date) live in the [Roadmap](roadmap.md) so they only need to be tracked in one place.

---

## 5. Technical Environment

### Development Tools

| Tool                      | Purpose                              |
| ------------------------- | ------------------------------------ |
| VS Code / Qoder IDE       | Code editor with TypeScript support  |
| Gitea (sdp.ms.wits.ac.za) | Primary Git host, code review, CI    |
| GitHub                    | Mirror for Vercel/Render auto-deploy |
| Trello                    | Sprint board and backlog             |
| Lovable                   | UI prototyping (see UI Design doc)   |
| Supabase Dashboard        | Database management, auth monitoring |

### Tech Stack (with justifications)

| Layer         | Technology            | Why                                                                              |
| ------------- | --------------------- | -------------------------------------------------------------------------------- |
| Frontend      | React 19 + Vite 8     | Component model for dashboard reuse; Vite provides fast HMR and optimised builds |
| Routing       | React Router 7        | Standard React SPA routing with nested routes for dashboards                     |
| Data fetching | TanStack React Query  | Server state caching, background refetch, optimistic updates                     |
| Icons         | Lucide React          | Tree-shakeable, consistent icon library                                          |
| Backend       | Express 5             | Widely used Node.js framework with large middleware ecosystem                    |
| ORM           | Prisma 7              | Type-safe queries, auto-generated types, version-controlled migrations           |
| Database      | PostgreSQL (Supabase) | ACID compliance for competition data; managed hosting with backups               |
| Auth          | Supabase Auth         | Managed JWT authentication, email/password flow, session management              |
| Security      | Helmet + CORS         | HTTP security headers and origin restriction                                     |
| Validation    | express-validator     | Declarative request validation with sanitisation                                 |
| File upload   | Multer                | Handles multipart/form-data for PDF paper uploads                                |
| Real-time     | ws (WebSocket)        | Live round state updates for organiser console                                   |
| Linting       | Oxlint                | Rust-based linter, 50-100x faster than ESLint                                    |
| TypeScript    | TypeScript 6/7        | Static types for a 6-person team working on shared code                          |

For deeper rationale behind each choice, see [Frontend](frontend.md), [Backend](backend.md), [Data and Storage](data-storage.md), and [Third-Party Code](third-party-code.md).

### Deployment

| Service  | Component         | Why                                                       |
| -------- | ----------------- | --------------------------------------------------------- |
| Render   | Backend API       | Free tier, auto-deploy from GitHub, automatic HTTPS       |
| Vercel   | Frontend SPA      | Purpose-built for React, instant preview deployments, CDN |
| Supabase | PostgreSQL + Auth | Managed database with backups, built-in auth service      |

---

## 6. Document History

| Version | Date           | Author               | Changes                  |
| ------- | -------------- | -------------------- | ------------------------ |
| 1.0     | September 2026 | Engedzani Mutambedzo | Initial development plan |