# 4. Data and Storage

## 4.1 Database — Supabase (PostgreSQL)

The database is PostgreSQL, hosted on Supabase. The data involved is relational in
a substantive way — schools, educators, entrants, rounds, submissions, and marks,
with constraints such as one submission per entrant per round — and a relational
database enforces this correctness rather than leaving it to application code.

Supabase is used strictly as **managed infrastructure**: Express connects to the
underlying PostgreSQL instance directly through Prisma, and the React application
never queries application data through Supabase's client library.

!!! note "Why this distinction matters"
    The Project Brief's Key Requirements name Supabase directly as an example of a
    system that generates API endpoints, which may not be used in place of a
    hand-written API. Used only as a managed database, authentication provider, and
    object store — with all application data routed exclusively through the
    hand-written Express API — this restriction is not expected to apply. The group
    intends to confirm this interpretation with the course lecturer ahead of
    Milestone 1.

## 4.2 Authentication — Supabase Auth

Authentication is handled by Supabase Auth. The React application calls it directly
for sign up, sign in, and password reset, and receives a JWT, which Express
verifies on every protected request. This satisfies the requirement that
authentication rely on an established library rather than a system written by the
group, and is a separate concern from the data-API restriction described above.

## 4.3 File Storage — Supabase Storage

Papers, memos, and scanned offline submissions are stored as objects in Supabase
Storage, accessed from Express using a service role key rather than by direct,
unauthenticated client access.