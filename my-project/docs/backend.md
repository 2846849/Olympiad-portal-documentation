# 3. Backend

## 3.1 Runtime and Framework — Node.js and Express

The backend is built on Node.js with Express. This shares a language with the
frontend, reducing context-switching across the group, and Express is minimally
opinionated — allowing the backend to be organised into the module structure set
out in [Architecture §1.2](architecture.md#12-backend-module-structure) rather than
one imposed by the framework. Express also integrates directly with the Node.js
job-scheduling libraries the round lifecycle requires.

## 3.2 API Style — REST

The API is a hand-written REST API, documented with OpenAPI. This satisfies the
requirement that the API be designed and implemented by the group rather than
generated, and is the most straightforward style to reason about across a group
with a range of prior experience.

## 3.3 Object-Relational Mapping — Prisma

Prisma is used as the ORM layer between Express and PostgreSQL. It provides type
safety consistent with a TypeScript backend, together with a migration workflow
suited to a group of six making frequent schema changes early in the project.