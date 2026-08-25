# 9. Version Control

## 9.1 Branching Strategy

The group uses **Git-flow**: a single `main` branch that is always deployable,
with short-lived feature branches merged in through reviewed pull requests. This
is simple enough for a group of six to sustain consistency.

## 9.2 Commit Convention

Commits follow the **Conventional Commits** format, enforced through a
commit-lint pre-commit hook.

## 9.3 Versioning

The project is versioned using **Semantic Versioning**, which communicates
backward compatibility to anything consuming the API.