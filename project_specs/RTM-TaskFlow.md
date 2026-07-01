# Requirements Traceability Matrix — TaskFlow

**Project Acronym:** TaskFlow  
**Version:** 1.0  
**Date:** 2026-07-01  
**Status:** Draft  
**Based On:** PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, TechArch-TaskFlow.md v1.0, UserStories-TaskFlow.md v1.0

---

## 1. Overview

This Requirements Traceability Matrix (RTM) provides bidirectional traceability between all TaskFlow specification documents. It ensures every product requirement defined in the PRD is elaborated in the FRD, backed by a concrete technical implementation in the TechArch, and validated by one or more user stories with associated acceptance criteria. Conversely, every user story and architectural decision can be traced back upstream to a specific product requirement, demonstrating that no implementation work exists without a business justification.

TaskFlow v1 is a lightweight shared task-tracking web application for small internal teams. Its six PRD features (F0–F5) span core CRUD operations — Task Creation, Task List View, Task Edit, Task Deletion, Data Persistence, and Status Filtering — all scoped to a single shared workspace with no authentication. Traceability is maintained across four specification levels: **PRD** (product features and priorities), **FRD** (functional behavior, validation rules, and API contracts), **TechArch** (component architecture, data model, security, and stack decisions), and **UserStories** (persona-driven acceptance criteria).

The RTM also defines the test case coverage framework, mapping each user story to a set of required test cases that verify both the happy path and error/edge-case behavior. This document serves as the authoritative traceability reference for all downstream activities: development, QA, code review, and change management.

---

## 2. Requirements Summary

### 2.1 PRD Features

- **F0 — Task Creation** (P0, Critical, MVP): Users can create a new task with a required title and optional description, status, and due date; task is immediately persisted and visible to all team members.
- **F1 — Task List View** (P0, Critical, MVP): Users can view all tasks in a sorted, paginated list with key fields at a glance; supports navigation to edit/detail views; renders empty state when no tasks exist.
- **F2 — Task Edit** (P0, Critical, MVP): Users can edit any field of an existing task with pre-populated form values; identical validation to creation; last-write-wins concurrency for v1.
- **F3 — Task Deletion** (P0, Critical, MVP): Users can permanently delete a task with a mandatory confirmation step; hard delete with no soft-delete in v1.
- **F4 — Data Persistence** (P0, Critical, MVP): All task data is stored in a relational database (PostgreSQL in production, SQLite in development); data survives server restarts; all writes are synchronous and confirmed before HTTP response.
- **F5 — Status Filtering** (P1, High, MVP): Users can filter the task list by status using URL-encoded query parameters; bookmarkable and session-persistent via URL; active filter visually indicated.

### 2.2 FRD Functional Requirement Chunks

- **F00** — Task Creation: form rendering, POST submission, server-side validation, DB persistence, success redirect, error re-render
- **F01** — Task List View: full list render, filtered list render, task row display, empty state, navigation to edit
- **F02** — Task Edit: edit form render (pre-populated), PUT submission, validation, DB update, redirect, 404/400 handling
- **F03** — Task Deletion: delete intent capture, confirmation step, DELETE execution, redirect, cancellation, 404/400 handling
- **F04** — Data Persistence: schema definition, ORM migration, synchronous writes, no in-memory storage, PostgreSQL/SQLite support
- **F05** — Status Filtering: filter control render, slug→enum mapping, URL encoding, active indicator, invalid-slug fallback, empty state
- **Y0** — Database Schema DDL (PostgreSQL + SQLite)
- **Y1** — REST API Endpoint Catalog (10 endpoints)
- **Y2** — Cross-Feature Error Catalog (11 error codes)
- **Y3** — External Integration Points (DB, browser, no third-party services)

### 2.3 TechArch Specification Areas

- **SPEC-ARCH** — Monolithic server-rendered architecture pattern; single process + single DB; stateless application server
- **SPEC-COMP** — Component architecture: Router, TaskController, Validation Layer, Task Model (ORM), Template Engine
- **SPEC-DATA** — Data model: single `tasks` table, PostgreSQL DDL, SQLite DDL, column definitions, indexes, query patterns
- **SPEC-API** — API design: TypeScript interfaces, 10 REST endpoints, full error code reference
- **SPEC-SEC** — Security architecture: no auth, input validation/sanitization, XSS output encoding, HTTP security headers, error information leakage prevention, DB connection security
- **SPEC-STACK** — Technology stack: Node.js 18+/Express 4.x/Sequelize 6.x/EJS (or Django 4.x/Django ORM/Jinja2); PostgreSQL 14+ (prod), SQLite 3.x (dev)
- **SPEC-INTG** — Integration points: DB integration, browser integration, no third-party services

### 2.4 User Stories

- **20 user stories** across 6 epics (US-0.1–US-5.4)
- **16 P0 stories** (US-0.1 through US-4.3) — must ship for MVP
- **4 P1 stories** (US-5.1 through US-5.4) — ship with MVP for usability
- **2 personas**: PER-01 Marcus Webb (Team Member), PER-02 Priya Nair (Team Lead)

### 2.5 Non-Functional Requirements Summary

- **Performance**: Task list page loads ≤ 2 seconds with up to 500 tasks
- **Reliability**: Zero data loss on server restart; synchronous DB writes confirmed before HTTP response
- **Security**: XSS prevention via output escaping; SQL injection prevention via ORM parameterized queries; HTTP security headers
- **Usability**: Core CRUD completable in ≤ 3 clicks from the task list
- **Accessibility**: Form inputs have `<label>` elements; status badges meet WCAG AA color contrast
- **Browser Support**: Chrome, Firefox, Safari, Edge (current versions)
- **Portability**: Runnable via single command after initial setup

---

## 3. Traceability Matrix

### 3.1 PRD Feature → FRD → TechArch → User Stories

| PRD Feature | FRD Chunk | TechArch Spec | User Stories | Priority |
|---|---|---|---|---|
| **F0: Task Creation** | F00 (Task Creation), Y0 (Schema), Y1 (POST /tasks, GET /tasks/new), Y2 (TITLE_REQUIRED, TITLE_TOO_LONG, DUE_DATE_INVALID, DB_WRITE_ERROR) | SPEC-COMP (TaskController.create, validateTaskInput, Task Model), SPEC-API (POST /tasks), SPEC-DATA (INSERT query pattern), SPEC-SEC (input validation/sanitization, XSS encoding) | US-0.1, US-0.2, US-0.3 | P0 |
| **F1: Task List View** | F01 (Task List View), Y0 (Schema), Y1 (GET /tasks), Y2 (DB_READ_ERROR) | SPEC-COMP (TaskController.list, tasks/index template), SPEC-API (GET /tasks), SPEC-DATA (SELECT unfiltered query), SPEC-ARCH (stateless server, DB reads on every request) | US-1.1, US-1.2, US-1.3 | P0 |
| **F2: Task Edit** | F02 (Task Edit), Y0 (Schema), Y1 (GET /tasks/:id/edit, PUT /tasks/:id), Y2 (TITLE_REQUIRED, TITLE_TOO_LONG, DUE_DATE_INVALID, STATUS_INVALID, TASK_NOT_FOUND, INVALID_TASK_ID, DB_WRITE_ERROR) | SPEC-COMP (TaskController.edit, TaskController.update, validateTaskInput), SPEC-API (GET /tasks/:id/edit, PUT /tasks/:id), SPEC-DATA (SELECT + UPDATE query patterns) | US-2.1, US-2.2, US-2.3, US-2.4 | P0 |
| **F3: Task Deletion** | F03 (Task Deletion), Y0 (Schema), Y1 (DELETE /tasks/:id, POST /tasks/:id/delete), Y2 (TASK_NOT_FOUND, INVALID_TASK_ID, DB_DELETE_ERROR) | SPEC-COMP (TaskController.destroy, Confirmation Dialog), SPEC-API (DELETE /tasks/:id), SPEC-DATA (DELETE query pattern), SPEC-SEC (hard delete, confirmation mitigates accidental loss) | US-3.1, US-3.2, US-3.3 | P0 |
| **F4: Data Persistence** | F04 (Data Persistence), Y0 (Schema DDL + indexes + migrations), Y3 (DB integration) | SPEC-DATA (full DDL, SQLite DDL, column definitions, indexes), SPEC-STACK (PostgreSQL 14+, SQLite 3.x, Sequelize/Django ORM), SPEC-INTG (DB connection, health check, migration flow), SPEC-ARCH (stateless app server, DB as sole state store) | US-4.1, US-4.2, US-4.3 | P0 |
| **F5: Status Filtering** | F05 (Status Filtering), Y1 (GET /tasks?status={slug}), Y2 (DB_READ_ERROR) | SPEC-COMP (TaskController.list with slug→enum mapping, filter controls in tasks/index template), SPEC-API (GET /tasks with ?status= query param, slug→enum mapping table), SPEC-DATA (SELECT filtered query pattern) | US-5.1, US-5.2, US-5.3, US-5.4 | P1 |

---

### 3.2 FRD Requirement → PRD Feature (Reverse Traceability)

| FRD Chunk | Maps To PRD Feature | Justification |
|---|---|---|
| F00 — Task Creation | F0 | Direct 1:1 elaboration of F0 capabilities |
| F01 — Task List View | F1 | Direct 1:1 elaboration of F1 capabilities |
| F02 — Task Edit | F2 | Direct 1:1 elaboration of F2 capabilities |
| F03 — Task Deletion | F3 | Direct 1:1 elaboration of F3 capabilities |
| F04 — Data Persistence | F4 | Direct 1:1 elaboration of F4 capabilities |
| F05 — Status Filtering | F5 | Direct 1:1 elaboration of F5 capabilities |
| Y0 — Database Schema | F0, F1, F2, F3, F4, F5 | Shared infrastructure; all features read/write the `tasks` table |
| Y1 — API Catalog | F0, F1, F2, F3, F5 | Each endpoint maps to one or more PRD features |
| Y2 — Error Catalog | F0, F1, F2, F3, F4 | Error codes cover all validation and infrastructure failure modes |
| Y3 — Integration Points | F4 (primary), F0–F5 (all) | DB integration underpins all persistence; browser integration covers all UI features |

---

### 3.3 User Story → FRD → PRD (Reverse Traceability)

| User Story | FRD Chunk(s) | PRD Feature | Acceptance Criteria Count |
|---|---|---|---|
| US-0.1: Create a New Task with a Title | F00 §Process steps 1–10, §Outputs (success) | F0 | 6 |
| US-0.2: Create a Task with Optional Fields | F00 §Inputs, §Validation Rules | F0 | 6 |
| US-0.3: See Validation Errors on Invalid Task Creation | F00 §Validation Rules, §Error States, Y2 (422 errors) | F0 | 6 |
| US-1.1: View All Tasks at a Glance | F01 §Process steps 1–9, §Outputs | F1 | 5 |
| US-1.2: Navigate from Task List to Detail/Edit View | F01 §Process step 6, Y1 (GET /tasks/:id/edit) | F1 | 4 |
| US-1.3: See an Empty State When No Tasks Exist | F01 §Empty State Messages | F1 | 4 |
| US-2.1: Open the Edit Form Pre-Populated | F02 §Process steps 1–4, §Inputs, §Error States | F2 | 5 |
| US-2.2: Save Changes to a Task | F02 §Process steps 5–12, §Outputs (success) | F2 | 6 |
| US-2.3: Update a Task's Status | F02 §Validation Rules (status on edit), §Error States (STATUS_INVALID) | F2 | 4 |
| US-2.4: See Validation Errors on Invalid Task Edit | F02 §Validation Rules, §Error States, Y2 (422 errors) | F2 | 6 |
| US-3.1: Delete a Task with a Confirmation Step | F03 §Process steps 1–9, §Outputs (confirmed deletion) | F3 | 6 |
| US-3.2: Cancel a Deletion and Return to Task List | F03 §Process step 3, §Outputs (cancellation) | F3 | 4 |
| US-3.3: Handle Deletion of a Non-Existent Task | F03 §Error States (TASK_NOT_FOUND, INVALID_TASK_ID) | F3 | 3 |
| US-4.1: Trust That Tasks Persist Across Server Restarts | F04 §Process step 4, §Validation Rules (no in-memory storage) | F4 | 4 |
| US-4.2: Set Up the Database Schema from Scratch | F04 §Sub-features, Y0 §Migration Script | F4 | 5 |
| US-4.3: Receive Confirmation That My Changes Were Saved | F04 §Process step 2, §Validation Rules (synchronous writes) | F4 | 4 |
| US-5.1: Filter the Task List by Status | F05 §Process steps 1–11, §Inputs | F5 | 5 |
| US-5.2: See the Active Filter Visually Indicated | F05 §Process step 4, §Validation Rules (URL-driven active filter) | F5 | 4 |
| US-5.3: Bookmark or Share a Filtered Task List URL | F05 §Outputs, §Validation Rules (bookmarkable filter) | F5 | 4 |
| US-5.4: See an Empty State When No Tasks Match the Filter | F05 §Sub-features (empty state), F01 §Empty State Messages | F5 | 5 |

---

## 4. Requirements Detail

### F0: Task Creation

**PRD Description:** Users can create a new task with a required title and optional description, status (default: To Do), and due date. Task is immediately persisted and visible to all team members.

**FRD Requirements (F00):**
- Form rendered at `GET /tasks/new` with title (text), description (textarea), status (dropdown, default: To Do), due date (date picker)
- `POST /tasks` persists to DB; `created_at` and `updated_at` set to current UTC timestamp
- Title validation: non-empty after trim, max 255 characters → errors: `TITLE_REQUIRED`, `TITLE_TOO_LONG`
- Due date validation: valid `YYYY-MM-DD` if provided → error: `DUE_DATE_INVALID`
- Status defaults to `To Do` if missing/invalid on creation (silent default, no error)
- On success: HTTP 302 redirect to `GET /tasks`; new task visible at top (newest-first)
- On failure: HTTP 422; form re-rendered with submitted values and inline error messages; all errors shown simultaneously

**TechArch Implementation:**
- `TaskController.create()`: extract + trim fields → `validateTaskInput(input, 'create')` → `Task.create({...})` → redirect
- `validateTaskInput()` returns `ValidationError[]`; empty array = valid
- `Task` model maps to `tasks` table; ORM parameterized insert
- EJS/Jinja2 auto-escaping on all template interpolations (XSS prevention)
- DB constraint: `title NOT NULL`, `status CHECK IN (...)` as defense-in-depth

---

### F1: Task List View

**PRD Description:** Shared task list displaying all tasks sorted by creation date (newest first), with status badges, due dates, and description previews. Links to edit views; shows empty state when no tasks exist.

**FRD Requirements (F01):**
- `GET /tasks` returns HTTP 200; queries `tasks` table `ORDER BY created_at DESC`
- Each task row: title (link to `GET /tasks/:id/edit`), status badge, due date formatted `MMM DD, YYYY` (blank if NULL), description preview (first 100 chars + `…` if truncated)
- Empty state (unfiltered): "No tasks yet. Create your first task to get started."
- Empty state (filtered): "No tasks with status '[Status Label]'. Try a different filter or create a new task."
- "New Task" button links to `GET /tasks/new`; visible even in empty state
- On DB error: HTTP 500 with `DB_READ_ERROR`

**TechArch Implementation:**
- `TaskController.list()`: reads `?status=` param → maps slug → `Task.findAll({where, order})` → renders `tasks/index`
- `tasks/index` template: filter control tabs (`All`, `To Do`, `In Progress`, `Done`), task rows, empty state message, "New Task" link
- Status badge: visually distinct chip/pill; must meet WCAG AA color contrast
- Performance: ≤ 2 seconds page load with up to 500 tasks on single server
- `idx_tasks_created_at` index supports `ORDER BY created_at DESC`

---

### F2: Task Edit

**PRD Description:** Users can edit any field of an existing task (title, description, status, due date) via a pre-populated form. Validation identical to creation. Last-write-wins concurrency for v1.

**FRD Requirements (F02):**
- `GET /tasks/:id/edit` renders edit form pre-populated with current task values; HTTP 404 if not found, HTTP 400 if `:id` not a positive integer
- `PUT /tasks/:id` (or `POST /tasks/:id` with `_method=PUT`) updates task; `updated_at` set to current UTC; `created_at` never modified
- Title validation: same as F00 → `TITLE_REQUIRED`, `TITLE_TOO_LONG`
- Status validation on edit: must be one of `To Do`, `In Progress`, `Done`; cannot be blank (unlike creation) → `STATUS_INVALID`
- Due date validation: same as F00 → `DUE_DATE_INVALID`
- Clearing description or due date saves NULL (no error)
- On success: HTTP 302 redirect to `GET /tasks`
- On failure: HTTP 422; form re-rendered with submitted (not original) values

**TechArch Implementation:**
- `TaskController.edit()`: parse `:id` → `Task.findByPk(id)` → render `tasks/edit` pre-populated
- `TaskController.update()`: parse `:id` → find task → `validateTaskInput(input, 'edit')` → `task.update({...})` → redirect
- `validateTaskInput()` context `'edit'` enforces `status` as required (unlike `'create'` where it defaults)
- Hidden `<input name="_method" value="PUT">` in HTML form for method override
- `idx_tasks_status` index supports filtered list after redirect

---

### F3: Task Deletion

**PRD Description:** Users can permanently delete a task with a mandatory confirmation step ("Are you sure you want to delete '[task title]'? This action cannot be undone."). Hard delete only for v1.

**FRD Requirements (F03):**
- Delete button/link available on task list row and/or task detail view
- Confirmation step is mandatory before `DELETE /tasks/:id` is sent; UX implementation choice: `window.confirm()` or lightweight vanilla JS modal
- `DELETE /tasks/:id` (or `POST /tasks/:id/delete` with `_method=DELETE`) hard-deletes the row
- On confirmation + success: HTTP 302 redirect to `GET /tasks`; task no longer appears
- On cancellation: no request sent; user returned to `GET /tasks`; no DB change
- On task not found: HTTP 404 with "Task not found. It may have already been deleted."
- On invalid `:id`: HTTP 400 with `INVALID_TASK_ID`
- On DB error: HTTP 500 with `DB_DELETE_ERROR`

**TechArch Implementation:**
- `TaskController.destroy()`: parse `:id` → `Task.findByPk(id)` → `task.destroy()` → redirect
- Confirmation dialog: Option A (recommended): `<form>` + `window.confirm()` onclick; Option B: vanilla JS modal
- No soft-delete column in v1 schema; `id` values never reused after deletion (SERIAL)
- Security: deletion without auth mitigated by confirmation dialog; soft-delete is v1.1 candidate

---

### F4: Data Persistence

**PRD Description:** All task data stored in a relational database; survives server restarts; all writes synchronous and confirmed before HTTP response; schema created via ORM migrations.

**FRD Requirements (F04):**
- `tasks` table DDL: `id SERIAL PK`, `title VARCHAR(255) NOT NULL`, `description TEXT NULL`, `status VARCHAR(20) NOT NULL DEFAULT 'To Do' CHECK(...)`, `due_date DATE NULL`, `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`, `updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`
- Migration command: `npx sequelize-cli db:migrate` (Node) or `python manage.py migrate` (Django)
- All writes (INSERT/UPDATE/DELETE) awaited before HTTP redirect/response
- No in-memory or session-scoped storage permitted
- PostgreSQL 14+ for production; SQLite 3.x for local dev/test; switch via `DATABASE_URL` only
- DB connection error on startup → health check failure + `DB_CONNECTION_ERROR` log
- Schema not initialized → `DB_SCHEMA_MISSING` (logged; generic message in prod)

**TechArch Implementation:**
- Single `tasks` table; no foreign keys; no user/session tables
- Indexes: `idx_tasks_status` (supports `WHERE status = $1`), `idx_tasks_created_at` (supports `ORDER BY created_at DESC`)
- ORM: Sequelize 6.x or Django ORM; all queries via ORM methods — no raw string-concatenated SQL
- `DATABASE_URL` environment variable; connection pool 5–10 (PostgreSQL)
- Health check query `SELECT 1` on startup; HTTP 503 if DB unreachable at request time
- `updated_at` auto-managed by ORM on every `save()`/`update()`; `created_at` never modified

---

### F5: Status Filtering

**PRD Description:** Users can filter the task list by status (All / To Do / In Progress / Done). Active filter visually indicated; URL encodes filter for bookmarking/sharing; invalid slugs silently fall back to "All".

**FRD Requirements (F05):**
- Filter controls rendered above task list at all times; four options: `All`, `To Do`, `In Progress`, `Done`
- Each option is a link to `GET /tasks?status={slug}` (or `GET /tasks` for "All")
- Slug→enum mapping: `todo` → `To Do`, `in_progress` → `In Progress`, `done` → `Done`, absent/`all`/unknown → no WHERE clause
- Active filter driven by URL parameter (not client-side state); refresh-safe
- Invalid/unknown `?status=` silently treated as "All"; no error shown
- Empty state when filtered list is empty: "No tasks with status '[Status Label]'. Try a different filter or create a new task."

**TechArch Implementation:**
- `TaskController.list()` reads `?status=` slug, maps to DB enum value, applies optional `WHERE status = $1`
- `tasks/index` template renders active filter tab with visual distinction (bold/filled style)
- Filter options are plain `<a href>` links — no JavaScript required
- `idx_tasks_status` index supports filtered queries efficiently
- URL state is sole source of filter truth; no server-side session required

---

## 5. Test Case Coverage Matrix

### 5.1 Coverage by Feature and Story

| Feature | Story ID | Story Title | Test Cases | Test Type | Coverage |
|---|---|---|---|---|---|
| **F0: Task Creation** | US-0.1 | Create a New Task with a Title | TEST-001, TEST-002, TEST-003 | Happy Path, Navigation, Visibility | ✅ |
| | US-0.2 | Create a Task with Optional Fields | TEST-004, TEST-005, TEST-006 | Optional Fields, Defaults, Null Storage | ✅ |
| | US-0.3 | See Validation Errors on Invalid Task Creation | TEST-007, TEST-008, TEST-009, TEST-010 | Validation, Error Display, HTTP 422 | ✅ |
| **F1: Task List View** | US-1.1 | View All Tasks at a Glance | TEST-011, TEST-012, TEST-013 | List Render, Performance, Sort Order | ✅ |
| | US-1.2 | Navigate to Task Detail/Edit View | TEST-014, TEST-015 | Navigation, Link Behavior | ✅ |
| | US-1.3 | See an Empty State When No Tasks Exist | TEST-016, TEST-017 | Empty State, HTTP 200 | ✅ |
| **F2: Task Edit** | US-2.1 | Open Edit Form Pre-Populated | TEST-018, TEST-019, TEST-020 | Pre-population, HTTP 404, HTTP 400 | ✅ |
| | US-2.2 | Save Changes to a Task | TEST-021, TEST-022, TEST-023 | Update, Timestamps, Null Clearing | ✅ |
| | US-2.3 | Update a Task's Status | TEST-024, TEST-025 | Status Update, STATUS_INVALID | ✅ |
| | US-2.4 | See Validation Errors on Invalid Task Edit | TEST-026, TEST-027, TEST-028 | Validation, Error Display, HTTP 422 | ✅ |
| **F3: Task Deletion** | US-3.1 | Delete a Task with a Confirmation Step | TEST-029, TEST-030, TEST-031 | Confirmation, Hard Delete, Redirect | ✅ |
| | US-3.2 | Cancel a Deletion | TEST-032, TEST-033 | Cancellation, No DB Change | ✅ |
| | US-3.3 | Handle Deletion of Non-Existent Task | TEST-034, TEST-035 | HTTP 404, HTTP 400 | ✅ |
| **F4: Data Persistence** | US-4.1 | Tasks Persist Across Server Restarts | TEST-036, TEST-037 | Restart Survival, No In-Memory Storage | ✅ |
| | US-4.2 | Set Up Database Schema from Scratch | TEST-038, TEST-039 | Migration, Schema Verification | ✅ |
| | US-4.3 | Receive Confirmation of Saved Changes | TEST-040, TEST-041 | Synchronous Write, DB Error → HTTP 500 | ✅ |
| **F5: Status Filtering** | US-5.1 | Filter the Task List by Status | TEST-042, TEST-043, TEST-044 | Filter by Each Status, Correct Results | ✅ |
| | US-5.2 | See Active Filter Visually Indicated | TEST-045, TEST-046 | Active Style, URL-Driven, Refresh-Safe | ✅ |
| | US-5.3 | Bookmark or Share a Filtered URL | TEST-047, TEST-048 | URL Encoding, Bookmarkable Load | ✅ |
| | US-5.4 | Empty State When No Tasks Match Filter | TEST-049, TEST-050, TEST-051 | Filtered Empty State, Invalid Slug Fallback | ✅ |

---

### 5.2 Test Case Definitions

| Test ID | Description | Story | Expected Result | Test Type |
|---|---|---|---|---|
| TEST-001 | Navigate to `GET /tasks/new`; verify form renders with title, description, status (default: To Do), due date fields | US-0.1 | HTTP 200; all four fields present; status pre-selected as "To Do" | Functional |
| TEST-002 | Submit `POST /tasks` with valid title; verify redirect and task visibility | US-0.1 | HTTP 302 → `GET /tasks`; new task at top of list | Functional |
| TEST-003 | Create a task; verify it appears immediately in the task list for all viewers (same-session check) | US-0.1 | Task visible on `GET /tasks` without additional steps | Functional |
| TEST-004 | Submit task with description, non-default status, and due date; verify all fields stored correctly | US-0.2 | DB row has correct title, description, status, due_date values | Functional |
| TEST-005 | Submit task without description and without due date; verify NULL storage | US-0.2 | DB row has `description = NULL`, `due_date = NULL`; no error shown | Functional |
| TEST-006 | Submit task without selecting status; verify default to "To Do" | US-0.2 | DB row has `status = 'To Do'`; no validation error | Functional |
| TEST-007 | Submit `POST /tasks` with empty title; verify HTTP 422 and error message | US-0.3 | HTTP 422; "Title is required." displayed inline | Validation |
| TEST-008 | Submit `POST /tasks` with title > 255 characters; verify HTTP 422 and error message | US-0.3 | HTTP 422; "Title must be 255 characters or fewer." displayed | Validation |
| TEST-009 | Submit `POST /tasks` with invalid due date (e.g., `2026-02-30`); verify HTTP 422 and error message | US-0.3 | HTTP 422; "Due date must be a valid date (YYYY-MM-DD)." displayed | Validation |
| TEST-010 | Submit form with both empty title and invalid due date; verify both errors shown simultaneously | US-0.3 | HTTP 422; both error messages displayed; submitted values pre-populated | Validation |
| TEST-011 | Navigate to `GET /tasks` with 3+ tasks; verify each row shows title, status badge, due date, description preview | US-1.1 | HTTP 200; all task rows rendered with correct fields | Functional |
| TEST-012 | Load `GET /tasks` with up to 500 tasks; verify page load within 2 seconds | US-1.1 | Response time ≤ 2000ms | Performance |
| TEST-013 | Create multiple tasks; verify list sorted newest first | US-1.1 | Tasks ordered by `created_at DESC` | Functional |
| TEST-014 | Click a task row title link; verify navigation to `GET /tasks/:id/edit` | US-1.2 | Browser navigates to edit form for that task ID | Functional |
| TEST-015 | Verify each task row has a clickable title link with correct `href` | US-1.2 | All task title links point to `/tasks/{id}/edit` | Functional |
| TEST-016 | Load `GET /tasks` with no tasks in DB; verify empty state message | US-1.3 | "No tasks yet. Create your first task to get started." displayed; HTTP 200 | Functional |
| TEST-017 | Verify "New Task" button is visible in empty state | US-1.3 | "New Task" link present and functional | Functional |
| TEST-018 | Navigate to `GET /tasks/:id/edit` for existing task; verify all fields pre-populated | US-2.1 | HTTP 200; form shows current title, description, status (selected), due date | Functional |
| TEST-019 | Navigate to `GET /tasks/999999/edit` (non-existent ID); verify HTTP 404 | US-2.1 | HTTP 404; "Task not found." message | Error Handling |
| TEST-020 | Navigate to `GET /tasks/abc/edit` (non-integer ID); verify HTTP 400 | US-2.1 | HTTP 400; "Invalid task ID." message | Error Handling |
| TEST-021 | Submit valid edit; verify redirect, updated values visible in task list, `updated_at` changed | US-2.2 | HTTP 302 → `GET /tasks`; DB row reflects new values; `updated_at` updated | Functional |
| TEST-022 | Edit a task; verify `created_at` is unchanged after save | US-2.2 | `created_at` value identical before and after edit | Functional |
| TEST-023 | Clear description and due date on edit; verify NULL saved | US-2.2 | DB row has `description = NULL`, `due_date = NULL` after save | Functional |
| TEST-024 | Change task status from "To Do" to "In Progress"; verify updated status badge visible in list | US-2.3 | DB `status = 'In Progress'`; correct badge shown on list | Functional |
| TEST-025 | Submit edit with invalid/blank status value; verify HTTP 422 and STATUS_INVALID error | US-2.3 | HTTP 422; "Status must be one of: To Do, In Progress, Done." displayed | Validation |
| TEST-026 | Submit edit with empty title; verify HTTP 422 and TITLE_REQUIRED error | US-2.4 | HTTP 422; "Title is required."; submitted values pre-populated | Validation |
| TEST-027 | Submit edit with title > 255 chars; verify HTTP 422 and TITLE_TOO_LONG error | US-2.4 | HTTP 422; "Title must be 255 characters or fewer." displayed | Validation |
| TEST-028 | Submit edit with multiple validation errors simultaneously; verify all shown | US-2.4 | HTTP 422; all applicable error messages displayed together | Validation |
| TEST-029 | Click Delete button; verify confirmation step appears with task title | US-3.1 | Confirmation dialog/page shows: "Are you sure you want to delete '[title]'? This action cannot be undone." | Functional |
| TEST-030 | Confirm deletion; verify task row removed from DB and task list | US-3.1 | HTTP 302 → `GET /tasks`; task no longer in DB or list | Functional |
| TEST-031 | Verify deleted task is not visible to any viewer after deletion | US-3.1 | `GET /tasks` does not include deleted task | Functional |
| TEST-032 | Click Cancel on confirmation dialog; verify no DELETE request sent | US-3.2 | No HTTP DELETE request sent; task remains in DB | Functional |
| TEST-033 | Cancel deletion; verify user returned to task list with task still visible | US-3.2 | `GET /tasks` shows task; no changes | Functional |
| TEST-034 | Send `DELETE /tasks/999999` (non-existent ID); verify HTTP 404 | US-3.3 | HTTP 404; "Task not found. It may have already been deleted." | Error Handling |
| TEST-035 | Send `DELETE /tasks/abc` (non-integer ID); verify HTTP 400 | US-3.3 | HTTP 400; "Invalid task ID." | Error Handling |
| TEST-036 | Create tasks, restart application server, reload `GET /tasks`; verify tasks still present | US-4.1 | All previously created tasks returned after restart | Functional |
| TEST-037 | Audit application code; verify no module-level arrays or Maps used to store task records | US-4.1 | No in-memory task storage present in source code | Compliance |
| TEST-038 | Run migration on empty DB; verify `tasks` table created with all required columns | US-4.2 | `tasks` table exists with correct schema; `status` CHECK constraint present | Functional |
| TEST-039 | Verify migration can be run with `DATABASE_URL` set to both PostgreSQL and SQLite without code changes | US-4.2 | Migration succeeds on both DB types; schema equivalent | Functional |
| TEST-040 | Simulate DB write failure; verify HTTP 500 returned and no partial state | US-4.3 | HTTP 500; "Unable to save the task. Please try again."; no redirect before write confirmed | Error Handling |
| TEST-041 | Simulate DB delete failure; verify HTTP 500 returned | US-4.3 | HTTP 500; "Unable to delete the task. Please try again." | Error Handling |
| TEST-042 | Navigate to `GET /tasks?status=todo`; verify only "To Do" tasks returned | US-5.1 | Task list contains only tasks with `status = 'To Do'` | Functional |
| TEST-043 | Navigate to `GET /tasks?status=in_progress`; verify only "In Progress" tasks returned | US-5.1 | Task list contains only tasks with `status = 'In Progress'` | Functional |
| TEST-044 | Navigate to `GET /tasks?status=done`; verify only "Done" tasks returned | US-5.1 | Task list contains only tasks with `status = 'Done'` | Functional |
| TEST-045 | Load `GET /tasks?status=in_progress`; verify "In Progress" filter option styled as active | US-5.2 | Active visual distinction applied to "In Progress" option only | Functional |
| TEST-046 | Load `GET /tasks` (no param); verify "All" filter shown as active | US-5.2 | "All" option shows active style; no other option is active | Functional |
| TEST-047 | Copy and reload a filtered URL (`/tasks?status=done`); verify same filtered view returned | US-5.3 | Reloaded URL returns correctly filtered task list | Functional |
| TEST-048 | Share filtered URL with second browser session; verify same filtered view | US-5.3 | Second session sees identical filtered list; filter state from URL only | Functional |
| TEST-049 | Apply "Done" filter with no completed tasks; verify filtered empty state message | US-5.4 | "No tasks with status 'Done'. Try a different filter or create a new task." displayed | Functional |
| TEST-050 | Verify filter controls and "New Task" link still visible in filtered empty state | US-5.4 | Filter tabs and "New Task" button present | Functional |
| TEST-051 | Navigate to `GET /tasks?status=invalid_value`; verify fallback to unfiltered "All" view | US-5.4 | All tasks returned; "All" shown as active; no error message | Error Handling |

---

### 5.3 Feature-Level Coverage Summary

| Feature | PRD Priority | User Stories | Test Cases | Happy Path | Error/Edge Cases | Coverage |
|---|---|---|---|---|---|---|
| F0: Task Creation | P0 | 3 (US-0.1–0.3) | 10 (TEST-001–010) | 6 | 4 | ✅ 100% |
| F1: Task List View | P0 | 3 (US-1.1–1.3) | 7 (TEST-011–017) | 6 | 1 | ✅ 100% |
| F2: Task Edit | P0 | 4 (US-2.1–2.4) | 11 (TEST-018–028) | 6 | 5 | ✅ 100% |
| F3: Task Deletion | P0 | 3 (US-3.1–3.3) | 7 (TEST-029–035) | 3 | 4 | ✅ 100% |
| F4: Data Persistence | P0 | 3 (US-4.1–4.3) | 6 (TEST-036–041) | 4 | 2 | ✅ 100% |
| F5: Status Filtering | P1 | 4 (US-5.1–5.4) | 10 (TEST-042–051) | 7 | 3 | ✅ 100% |
| **Total** | — | **20** | **51** | **32** | **19** | **✅ 100%** |

---

### 5.4 Non-Functional Requirements Test Coverage

| NFR Category | Requirement | Test ID | Verification Method |
|---|---|---|---|
| Performance | Task list ≤ 2 seconds with 500 tasks | TEST-012 | Load test with 500-task dataset |
| Reliability | Data survives server restart | TEST-036 | Integration test: create → restart → verify |
| Reliability | Synchronous writes before redirect | TEST-040, TEST-041 | Simulated DB failure injection |
| Security | XSS prevention via output escaping | Code review | Template audit: all user content uses escaped interpolation |
| Security | SQL injection prevention via ORM | Code review | Static analysis: no raw string-concatenated SQL |
| Usability | CRUD completable in ≤ 3 clicks | Manual test | UX walkthrough: create, edit, delete each from task list |
| Accessibility | Form labels and WCAG AA color contrast | Manual test / a11y scan | `axe` or similar accessibility audit |
| Browser Support | Works in Chrome, Firefox, Safari, Edge | Manual / automated cross-browser | Smoke test suite on each browser |
| Portability | Runs via single command | TEST-038, TEST-039 | `npm start` / `python manage.py runserver` from clean install |

---

## 6. API Endpoint Traceability

| # | Method | Path | FRD Section | PRD Feature | User Stories | TechArch Component |
|---|---|---|---|---|---|---|
| 1 | GET | `/` | Y1 §Root Redirect | — | — | `TaskController` (redirect) |
| 2 | GET | `/tasks` | Y1 §Task List, F01 | F1, F5 | US-1.1, US-1.2, US-1.3, US-5.1–5.4 | `TaskController.list` + `tasks/index` template |
| 3 | GET | `/tasks/new` | Y1 §Task Creation, F00 | F0 | US-0.1, US-0.2 | `TaskController.new` + `tasks/new` template |
| 4 | POST | `/tasks` | Y1 §Task Creation, F00 | F0 | US-0.1, US-0.2, US-0.3 | `TaskController.create` + `validateTaskInput` |
| 5 | GET | `/tasks/:id` | Y1 §Task Detail | F1, F2 | US-1.2 | `TaskController.show` (redirects to edit) |
| 6 | GET | `/tasks/:id/edit` | Y1 §Task Edit, F02 | F2 | US-2.1, US-2.2, US-2.3, US-2.4 | `TaskController.edit` + `tasks/edit` template |
| 7 | PUT | `/tasks/:id` | Y1 §Task Edit, F02 | F2 | US-2.2, US-2.3, US-2.4 | `TaskController.update` + `validateTaskInput` |
| 8 | POST | `/tasks/:id` | Y1 §Task Edit, F02 | F2 | US-2.2, US-2.3, US-2.4 | `TaskController.update` (method override) |
| 9 | DELETE | `/tasks/:id` | Y1 §Task Deletion, F03 | F3 | US-3.1, US-3.2, US-3.3 | `TaskController.destroy` |
| 10 | POST | `/tasks/:id/delete` | Y1 §Task Deletion, F03 | F3 | US-3.1, US-3.2, US-3.3 | `TaskController.destroy` (method override) |

---

## 7. Error Code Traceability

| Error Code | HTTP Status | FRD Source | PRD Feature | User Stories | Test Cases |
|---|---|---|---|---|---|
| `TITLE_REQUIRED` | 422 | Y2, F00 §Errors, F02 §Errors | F0, F2 | US-0.3, US-2.4 | TEST-007, TEST-026 |
| `TITLE_TOO_LONG` | 422 | Y2, F00 §Errors, F02 §Errors | F0, F2 | US-0.3, US-2.4 | TEST-008, TEST-027 |
| `DUE_DATE_INVALID` | 422 | Y2, F00 §Errors, F02 §Errors | F0, F2 | US-0.3, US-2.4 | TEST-009, TEST-026 |
| `STATUS_INVALID` | 422 | Y2, F02 §Errors | F2 | US-2.3, US-2.4 | TEST-025, TEST-028 |
| `INVALID_TASK_ID` | 400 | Y2, F02 §Errors, F03 §Errors | F2, F3 | US-2.1, US-3.3 | TEST-020, TEST-035 |
| `TASK_NOT_FOUND` | 404 | Y2, F02 §Errors, F03 §Errors | F2, F3 | US-2.1, US-3.3 | TEST-019, TEST-034 |
| `DB_READ_ERROR` | 500 | Y2, F01 §Errors, F05 §Errors | F1, F5 | US-1.1, US-5.1 | (Infrastructure test) |
| `DB_WRITE_ERROR` | 500 | Y2, F00 §Errors, F02 §Errors, F04 §Errors | F0, F2, F4 | US-4.3 | TEST-040 |
| `DB_DELETE_ERROR` | 500 | Y2, F03 §Errors | F3 | US-4.3 | TEST-041 |
| `DB_CONNECTION_ERROR` | 503 | Y2, F04 §Errors | F4 | US-4.1, US-4.3 | (Infrastructure test) |
| `DB_SCHEMA_MISSING` | 500 | Y2, F04 §Errors | F4 | US-4.2 | TEST-038 |

---

## 8. Schema Column Traceability

| Column | FRD Source | PRD Feature(s) | User Stories | TechArch Spec |
|---|---|---|---|---|
| `id` | Y0, F04 §Schema | F0, F1, F2, F3, F4 | All (implicit) | SPEC-DATA §3.1, SPEC-SEC §5.2 |
| `title` | Y0, F00 §Inputs, F02 §Inputs | F0, F1, F2 | US-0.1, US-0.2, US-0.3, US-2.2, US-2.4 | SPEC-DATA §3.4, SPEC-SEC §5.2 |
| `description` | Y0, F00 §Inputs, F02 §Inputs | F0, F1, F2 | US-0.2, US-2.2 | SPEC-DATA §3.4 |
| `status` | Y0, F00 §Inputs, F02 §Inputs, F05 §Slug Mapping | F0, F1, F2, F5 | US-0.2, US-2.3, US-5.1–5.4 | SPEC-DATA §3.4, SPEC-SEC §5.2 |
| `due_date` | Y0, F00 §Inputs, F02 §Inputs | F0, F1, F2 | US-0.2, US-0.3, US-2.2, US-2.4 | SPEC-DATA §3.4 |
| `created_at` | Y0, F04 §Schema | F0, F1, F4 | US-1.1 (sort order), US-2.2, US-4.1 | SPEC-DATA §3.4 |
| `updated_at` | Y0, F02 §Process, F04 §Schema | F2, F4 | US-2.2 | SPEC-DATA §3.4 |

---

## 9. Out-of-Scope Traceability

The following items are explicitly excluded from v1 across all specification documents and must not appear in any implementation, test, or architecture artifact:

| Item | PRD §10 | FRD Scope | TechArch §7.4 | Disposition |
|---|---|---|---|---|
| User Authentication / Login | ✅ Excluded | ✅ Out of scope | ✅ Not a third-party integration | v2+ candidate |
| Multi-tenancy / Per-user Isolation | ✅ Excluded | ✅ Out of scope | ✅ Not in schema | v2+ candidate |
| Real-time Collaboration / WebSockets | ✅ Excluded | ✅ Out of scope | ✅ No WebSocket integration | v2+ candidate |
| File Attachments | ✅ Excluded | ✅ Out of scope | ✅ Not in schema | Out of scope |
| Mobile Native App | ✅ Excluded | ✅ Out of scope | ✅ Not in integration points | Web-first |
| Task Assignment / Ownership | ✅ Excluded | ✅ Out of scope | ✅ No `assigned_to` column | v2+ (requires auth) |
| Comments / Activity Log | ✅ Excluded | ✅ Out of scope | ✅ No comments table | v2+ candidate |
| Soft Delete | ✅ Not mentioned | ✅ Hard delete only | ✅ No `deleted_at` column | v1.1 candidate |
| Optimistic Locking | ✅ Excluded (risk item) | ✅ Last-write-wins documented | ✅ Not in architecture | v2+ candidate |

---

## 10. Change Management

### 10.1 Change Log

| Version | Date | Author | Change Description | Affected Sections |
|---|---|---|---|---|
| 1.0 | 2026-07-01 | — | Initial RTM generated from PRD v1.0, FRD v1.0, TechArch v1.0, UserStories v1.0 | All |

### 10.2 Change Control Process

Changes to any linked specification document require a corresponding update to this RTM to maintain bidirectional traceability. The following change types trigger an RTM update:

- **New PRD feature** → Add new row to Section 3.1; add requirements detail in Section 4; add test cases in Section 5
- **FRD requirement change** → Update affected rows in Sections 3.1, 3.2, 3.3; update Section 4; update error catalog in Section 7
- **TechArch spec change** → Update affected rows in Sections 3.1 and 6
- **New/modified user story** → Update Section 3.3; update test cases in Section 5.2; update coverage summary in Section 5.3
- **Out-of-scope addition** → Update Section 9

### 10.3 Impact Assessment

When a change is proposed, assess impact across all four traceability levels before approving:

1. **PRD impact**: Does the change alter product scope, priorities, or success metrics?
2. **FRD impact**: Does the change require new validation rules, API endpoints, or error codes?
3. **TechArch impact**: Does the change affect the data model, security posture, or component design?
4. **UserStory impact**: Do existing acceptance criteria still hold, or are new stories required?

---

## 11. Approval

### 11.1 Document Sign-Off

| Role | Name | Signature | Date | Status |
|---|---|---|---|---|
| Product Owner | — | — | — | ⬜ Pending |
| Tech Lead / Architect | — | — | — | ⬜ Pending |
| QA Lead | — | — | — | ⬜ Pending |
| Development Lead | — | — | — | ⬜ Pending |

### 11.2 Review Checklist

Prior to approval, reviewers should confirm all of the following:

- [ ] All 6 PRD features (F0–F5) have at least one FRD chunk, one TechArch spec, and one user story in the traceability matrix
- [ ] All 20 user stories are mapped to at least one PRD feature and one FRD chunk
- [ ] All 51 test cases are assigned to a user story and have a defined expected result
- [ ] All 11 error codes in Y2 appear in Section 7 with correct HTTP status, feature mapping, and test reference
- [ ] All 7 `tasks` table columns appear in Section 8 with correct FRD source and feature mapping
- [ ] All out-of-scope items in PRD §10 appear in Section 9 with consistent exclusion across all spec documents
- [ ] No test case, user story, or architectural spec references a feature outside the v1 scope boundary

---

*Document generated: 2026-07-01 | RTM-TaskFlow v1.0*  
*Based on: PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, TechArch-TaskFlow.md v1.0, UserStories-TaskFlow.md v1.0, PROJECT.md v1.0*
