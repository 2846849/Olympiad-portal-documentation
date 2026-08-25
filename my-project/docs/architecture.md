# 1. Architecture

## 1.1 Overview

The system follows a client-server, three-tier architecture:

- **Presentation tier** — a React single-page application
- **Application tier** — a Node.js and Express REST API, structured internally as a modular monolith
- **Data tier** — a PostgreSQL database hosted on Supabase

The React application is the only client of the Express API; every read and write of
application data passes through hand-written Express endpoints.

The one exception is **authentication**: React calls Supabase Auth directly to sign
users in, sign them up, and reset passwords, receiving a JSON Web Token (JWT) in
return. Express then verifies that JWT on every protected request.

Express itself never calls Supabase's data API. It connects to the underlying
PostgreSQL database directly through Prisma, and to Supabase Storage directly for
file operations.

## 1.2 Backend Module Structure

The Express backend is organised into modules aligned with the project's feature
tiers, so that Intermediate and Advanced tier features can be added without
restructuring what already exists for the Basic tier.

| Module | Tier | Purpose |
|---|---|---|
| `auth` | Basic | Role verification middleware; validates the JWT issued by Supabase Auth |
| `schools` | Basic | School, educator, and entrant registration |
| `rounds` | Basic | Round creation and management; the round state machine (open, running, closed) |
| `papers` | Basic | Paper and memo upload, archive, and release gating based on round state |
| `submissions` | Basic | Result upload, receipt generation, multiple-choice automatic marking |
| `scheduling` | Basic | Automated round opening and closing; notification triggers |
| `notifications` | Basic; extended in Intermediate | EmailJS integration: receipts in the Basic tier, reminders and follow-ups from the Intermediate tier |
| `marking` | Intermediate | Human marking queue, partial credit, moderation, remarks |
| `standings` | Intermediate | Per-school and per-entrant results, certificates |
| `progression` | Intermediate | Qualifying thresholds and automatic advancement between rounds |
| `questionBank` | Advanced | Question variants and difficulty derived from response data |
| `appeals` | Advanced | Appeal routing to a reviewer and outcome propagation to standings |
| `tenancy` | Advanced | Multiple concurrent olympiads, each on its own schedule |

## 1.3 Database Schema Design Considerations

Several schema decisions were made up front specifically so that later tiers do not
require breaking changes:

- **Submission status** is stored as a string field rather than a boolean, so values
  such as *pending review*, *under moderation*, and *appealed* can be introduced
  later without a migration that breaks existing data.
- **Entrants are linked to rounds through a join table** rather than being embedded
  directly on the round record. This is what allows automatic advancement between
  rounds to be implemented as a data operation rather than a schema change.
- **Rounds include a nullable time zone field** from the outset, even though the
  Basic tier can assume a single time zone, so the Advanced tier's multi-time-zone
  requirement doesn't require retrofitting.
- **Papers and memos are kept independent** of any future question bank or variant
  table, so variants can be introduced later without altering the Basic tier's
  paper model.
- **The roles table reserves `marker` and `reviewer` roles** from the outset, even
  though only `organiser`, `educator`, and `student` roles are used in the Basic
  tier — so the Intermediate tier's marking queue and the Advanced tier's appeals
  process don't require a schema change to introduce them.