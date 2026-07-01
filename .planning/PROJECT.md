# TaskFlow

## What This Is

TaskFlow is a simple task tracking web application for small internal teams. Team members share a single workspace where they can create, view, edit, and delete tasks. Each task carries a title, optional description, status (To Do / In Progress / Done), and an optional due date, with all data persisted in a relational database.

## Core Value

Team members can reliably track the status of work across the team with a shared, persistent task list that survives restarts and requires zero login friction.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] User can create a new task with title, optional description, status, and optional due date
- [ ] User can view a list of all tasks, filterable by status
- [ ] User can edit an existing task (any field)
- [ ] User can delete a task with a confirmation step
- [ ] Tasks are persisted in a relational database (data survives server restart)
- [ ] No user authentication or multi-tenancy — single shared workspace

### Out of Scope

- User authentication / login — not needed for v1, single shared team workspace
- Multi-tenancy / per-user isolation — v2+, adds complexity without value for a small internal team
- Real-time collaboration / live updates — v2+, deferred complexity
- File attachments on tasks — out of scope, not mentioned in requirements
- Mobile native app — web-first; browser is sufficient for internal tooling

## Context

- Target users: small internal team (~2–15 people), no dedicated ops staff
- Usage pattern: lightweight daily check-in / status tracking, not a heavyweight project management tool
- Tech philosophy: "clean, working CRUD example" — conventional stack, no exotic dependencies, easy to understand and maintain
- Tech stack guidance: standard web framework + relational database (e.g., Express/Node or Django/Python + PostgreSQL/SQLite); keep it simple and conventional

## Constraints

- **Tech Stack**: Standard web framework + relational database — keep conventional and easy to maintain
- **Auth**: None required for v1 — single shared workspace, no per-user sessions
- **Persistence**: Real database (not in-memory) — data must survive server restarts
- **Scope**: CRUD-only for v1 — not a feature-rich product management tool

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| No authentication | Single shared team workspace; login adds friction without value for v1 | — Pending |
| Relational database (not in-memory) | Data must survive restarts; conventional choice for CRUD apps | — Pending |
| Status enum: To Do / In Progress / Done | Simple, universal states; avoids over-engineering for v1 | — Pending |

---
*Last updated: 2026-07-01 after initialization*
