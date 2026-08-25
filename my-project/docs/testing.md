# 7. Testing

The project currently uses **Vitest** as its sole testing tool, across both the
backend and frontend, run in continuous integration on every pull request.

**Why Vitest:** the frontend is built with Vite (see [Frontend §2.1](frontend.md#21-framework-react)),
and Vitest is designed to share Vite's configuration, transforms, and dev-server
pipeline directly — so no separate Jest/Babel setup is needed alongside the Vite
build. Using the same runner for the backend keeps a single test tool and config
style across the whole stack, rather than the group maintaining two different
testing setups and mental models. Vitest's API is also Jest-compatible, so it
doesn't introduce an unfamiliar testing paradigm for anyone on the team.

## 7.1 Backend Testing

Backend tests are written in Vitest, covering the API endpoints directly.

## 7.2 Frontend Testing

Frontend tests are written in Vitest at the component level.

!!! note "Not yet in use"
    Supertest, React Testing Library, and Playwright are not currently part of the
    test suite. As the project takes on the Intermediate and Advanced tiers —
    particularly the submission-deadline and automatic-marking flows — end-to-end
    coverage with a tool such as Playwright may be added; this section will be
    updated if and when that happens.