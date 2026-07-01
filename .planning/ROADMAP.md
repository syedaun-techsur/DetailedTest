# Roadmap: TaskFlow

## Overview

TaskFlow delivers a shared, persistent task tracker in three natural phases: first the foundational infrastructure (project scaffolding, database, ORM, migrations), then the core CRUD loop that makes the app actually useful (create, view, edit, delete tasks), and finally status filtering that lets the team focus on what matters. Each phase leaves users with something coherent and verifiable before the next begins.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation** - Project scaffolding, database schema, ORM setup, and migrations
- [ ] **Phase 2: Core CRUD** - Create, view, edit, and delete tasks with full persistence
- [ ] **Phase 3: Status Filtering** - URL-driven status filter with bookmarkable filtered views

## Phase Details

### Phase 1: Foundation
**Goal**: The application runs, connects to the database, and the schema is in place — ready for feature work
**Depends on**: Nothing (first phase)
**Requirements**: TASK-05
**Success Criteria** (what must be TRUE):
  1. Running `npm start` (or equivalent) starts the server without errors
  2. Running the migration command creates the `tasks` table with all required columns and indexes
  3. The app connects to both PostgreSQL (production) and SQLite (development) via `DATABASE_URL` alone — no code changes required
  4. Stopping and restarting the server does not affect data already in the database
**Plans**: 2 plans

Plans:
- [ ] 01-01-PLAN.md — Project scaffold: npm init, Express app, middleware, stub routes, error templates
- [ ] 01-02-PLAN.md — Database layer: Sequelize config, Task model (exact DDL), versioned migration

### Phase 2: Core CRUD
**Goal**: Team members can fully manage tasks — create, view, edit, and delete — with all changes persisted across restarts
**Depends on**: Phase 1
**Requirements**: TASK-01, TASK-02, TASK-03, TASK-04
**Success Criteria** (what must be TRUE):
  1. User can create a task with a title (required), optional description, status defaulting to "To Do", and optional due date — new task appears at the top of the list immediately
  2. User can view all tasks sorted by creation date (newest first), with title, status badge, due date, and description preview visible
  3. User can edit any field of an existing task and see the updated values reflected on the list after saving
  4. User can delete a task after confirming the action in a confirmation dialog — the task disappears from the list for all viewers
**Plans**: TBD

### Phase 3: Status Filtering
**Goal**: Team members can narrow the task list to a specific status, with the active filter encoded in the URL so filtered views can be bookmarked and shared
**Depends on**: Phase 2
**Requirements**: FILT-01
**Success Criteria** (what must be TRUE):
  1. User can filter the task list by clicking All / To Do / In Progress / Done controls — only matching tasks are shown
  2. The active filter is visually distinguished from inactive filters
  3. The URL reflects the active filter (e.g., `?status=in_progress`) — navigating to that URL directly shows the same filtered view
  4. An appropriate empty-state message is shown when no tasks match the active filter
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 0/2 | Not started | - |
| 2. Core CRUD | 0/? | Not started | - |
| 3. Status Filtering | 0/? | Not started | - |
