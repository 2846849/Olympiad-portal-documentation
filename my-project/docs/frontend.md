# 2. Frontend

## 2.1 Framework — React

The frontend is a React single-page application, built with Vite. React was chosen
because the project requires two functionally distinct interfaces — an organiser
console and an educator/student portal — both of which are dynamic and stateful,
involving round countdowns, live submission status, and marking queues. React's
component model divides this work cleanly across a group of six, and its ecosystem
provides supporting components (timers, data tables) that these interfaces require.

## 2.2 State Management — TanStack Query

Most of the application's state is **server state** — round status, submissions,
and marks — rather than local UI state. TanStack Query manages this, providing
caching and refetching without the extra boilerplate of a general-purpose state
management library.

## 2.3 Routing — React Router

React Router provides client-side routing, with routes gated by role so that the
organiser console and the educator/student portal can be served from a single
application.