# Functional Requirements Document — TaskFlow

**Project Acronym:** TaskFlow
**Version:** 1.0
**Date:** 2026-07-01
**Status:** Draft
**Based On:** PRD-TaskFlow.md v1.0, PROJECT.md v1.0

---

## Scope

This document specifies the complete functional behavior of TaskFlow v1 — a lightweight, shared task-tracking web application for small internal teams of 2–15 people. It covers all six PRD features (F0–F5), the relational database schema, the REST API endpoint catalog, the cross-feature error catalog, and external integration points. Downstream agents (code generators, test generators, reviewers) should treat this document as the single authoritative specification for what the system must do.

Authentication, multi-tenancy, real-time updates, file attachments, and task assignment are **explicitly out of scope for v1** and must not be implemented.

---

## Conventions

- **Feature IDs:** F00–F05 map 1:1 to PRD features F0–F5.
- **Field types:** SQL-style types used throughout (VARCHAR, TEXT, TIMESTAMP, etc.).
- **Required vs Optional:** `(required)` and `(optional)` annotations on all input fields.
- **HTTP verbs:** Standard REST semantics — GET (read), POST (create), PUT/PATCH (update), DELETE (remove).
- **Error codes:** Uppercase snake-case strings (e.g., `TITLE_REQUIRED`).
- **Enum values:** Status values are always one of exactly: `To Do`, `In Progress`, `Done` (display form) or `todo`, `in_progress`, `done` (URL/query-param form).
- **Timestamps:** All timestamps stored and returned in UTC ISO 8601 format (`YYYY-MM-DDTHH:MM:SSZ`).
- **Date-only fields:** `due_date` stored as `DATE` (no time component). Accepted input format: `YYYY-MM-DD`.
- **Cross-references:** `see F03 §Process` means refer to feature chunk F03, Process section.

---

## Master Table of Contents

| Section | File | Description |
|---------|------|-------------|
| F00 | `F00-task-creation.md` | Task Creation |
| F01 | `F01-task-list-view.md` | Task List View |
| F02 | `F02-task-edit.md` | Task Edit |
| F03 | `F03-task-deletion.md` | Task Deletion |
| F04 | `F04-data-persistence.md` | Data Persistence |
| F05 | `F05-status-filtering.md` | Status Filtering |
| Y0 | `Y0-schema.md` | Database Schema (DDL) |
| Y1 | `Y1-api.md` | REST API Endpoint Catalog |
| Y2 | `Y2-errors.md` | Cross-Feature Error Catalog |
| Y3 | `Y3-integrations.md` | External Integration Points |

---

## Cross-Cutting Terminology

- **Task:** The core domain entity. Represents a unit of work with a title, optional description, status, optional due date, and system-managed timestamps.
- **Shared Workspace:** The single, globally shared context in which all tasks exist. There is no per-user, per-team, or per-project isolation in v1.
- **Status:** An enum field on a task. Valid values: `To Do` (default), `In Progress`, `Done`.
- **Status slug:** The URL-safe, lowercase version of a status value used in query parameters: `todo`, `in_progress`, `done`.
- **Due Date:** An optional calendar date (no time) by which a task is intended to be completed.
- **Task List:** The primary view — a server-rendered page showing all tasks in the shared workspace, optionally filtered by status.
- **Task Detail / Edit View:** A server-rendered page showing a single task's full details and providing an edit form.
- **Confirmation Step:** A user-facing dialog or dedicated page that asks the user to explicitly confirm a destructive action (deletion) before it is executed.
- **ORM:** Object-Relational Mapper. TaskFlow uses Sequelize (Node.js) or Django ORM (Python) to issue parameterized queries; raw SQL is not used directly.
- **Migration:** A versioned, repeatable script that creates or alters the database schema. Required for initial setup and future schema changes.
- **Empty State:** The UI condition where a list contains zero items, requiring a specific message to be displayed instead of a blank list.

---

## Non-Functional Constraints (Summary)

| Category | Requirement |
|----------|-------------|
| Performance | Task list page loads ≤ 2 seconds on single-server with up to 500 tasks |
| Reliability | Zero data loss on server restart; DB writes synchronous and confirmed before HTTP response |
| Security | XSS prevention via output escaping; SQL injection prevention via ORM parameterized queries |
| Accessibility | Form inputs have `<label>` elements; status badges meet WCAG AA color contrast |
| Browser Support | Chrome, Firefox, Safari, Edge (current versions) |
| Usability | Core CRUD completable in ≤ 3 clicks from the task list |
| Portability | Runnable via single command after initial setup (`npm start` or `python manage.py runserver`) |

---

*End of header — see feature chunks F00–F05 and cross-feature chunks Y0–Y3 for full specification.*
---

## F00: Task Creation

**Description:** This feature allows any team member to create a new task in the shared workspace by submitting a form with a title and optional fields. The task is immediately written to the relational database and becomes visible to all team members the next time they load or refresh the task list. Task creation is the entry point for all work tracked in TaskFlow; without it no other features have data to operate on.

---

### Terminology

- **Creation Form:** The HTML form rendered at `GET /tasks/new` (or equivalent) that accepts task field inputs.
- **Default Status:** The status value automatically assigned when the user does not explicitly choose one — always `To Do`.
- **Title:** The mandatory, human-readable name of the task. Must be non-empty after trimming whitespace.
- **Description:** An optional, free-form multi-line text field providing additional context for the task.
- **Due Date:** An optional calendar date (`YYYY-MM-DD`) representing the intended completion deadline.

---

### Sub-features

- Render the task creation form (`GET /tasks/new`)
- Submit the task creation form (`POST /tasks`)
- Server-side validation of submitted fields
- Persist validated task to the database
- Redirect to task list on success with new task visible
- Re-render form with error messages on validation failure

---

### Process

1. User navigates to the task creation form (clicks "New Task" button or equivalent link from the task list).
2. Server renders the creation form with: title field (text input), description field (textarea), status dropdown (options: `To Do`, `In Progress`, `Done`; default selected: `To Do`), due date field (date input).
3. User fills in fields. Title is required; all other fields are optional.
4. User submits the form (`POST /tasks`).
5. Server trims whitespace from all text inputs.
6. Server validates inputs (see §Validation below).
7. **If validation fails:** Server re-renders the creation form, pre-populated with the submitted values, with inline error messages adjacent to each invalid field. HTTP 422 returned (or equivalent redirect with flash errors for server-rendered apps).
8. **If validation passes:** Server inserts a new row into the `tasks` table with `created_at` and `updated_at` set to the current UTC timestamp.
9. Server responds with HTTP 302 redirect to `GET /tasks` (task list).
10. Task list renders with the newly created task visible at the top (newest-first sort order).

---

### Inputs

- `title` (VARCHAR(255), required): Human-readable task name. Must be non-empty after whitespace trim. Maximum 255 characters.
- `description` (TEXT, optional): Additional context. Multi-line. No maximum length enforced at application level (DB TEXT type). Defaults to NULL if not provided.
- `status` (ENUM, optional): Task state. One of `To Do`, `In Progress`, `Done`. Defaults to `To Do` if not provided or if the submitted value is not one of the valid options.
- `due_date` (DATE string `YYYY-MM-DD`, optional): Intended completion date. Defaults to NULL if not provided.

---

### Outputs

**On success:**
- HTTP 302 redirect to `GET /tasks`.
- New `tasks` row with fields: `id` (system-assigned), `title`, `description` (or NULL), `status` (`To Do` default), `due_date` (or NULL), `created_at` (UTC now), `updated_at` (UTC now).

**On validation failure:**
- HTTP 422 (Unprocessable Entity) or server-rendered form page with errors.
- Form fields re-populated with submitted values.
- One or more inline error messages displayed.

---

### Validation Rules

- `title`: Must not be empty or whitespace-only after trimming. Maximum 255 characters. Violation → error message: `"Title is required."` / `"Title must be 255 characters or fewer."`
- `status`: If provided, must be exactly one of: `To Do`, `In Progress`, `Done`. If missing or invalid, silently default to `To Do` (no error shown to user).
- `due_date`: If provided, must be a valid calendar date in `YYYY-MM-DD` format. Must not be a non-existent date (e.g., `2026-02-30`). Violation → error message: `"Due date must be a valid date (YYYY-MM-DD)."`
- `description`: No validation constraints beyond storage (TEXT). Accepted as-is.

---

### Error States

| Scenario | HTTP Status | Error Code | User-Facing Message |
|----------|-------------|------------|---------------------|
| Title is empty or whitespace | 422 | `TITLE_REQUIRED` | "Title is required." |
| Title exceeds 255 characters | 422 | `TITLE_TOO_LONG` | "Title must be 255 characters or fewer." |
| `due_date` is not a valid date | 422 | `DUE_DATE_INVALID` | "Due date must be a valid date (YYYY-MM-DD)." |
| Database write fails | 500 | `DB_WRITE_ERROR` | "Unable to save the task. Please try again." |

---

### API Surface (this feature)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/tasks/new` | Render task creation form |
| `POST` | `/tasks` | Submit new task; persist to DB |

Full request/response schemas: see `Y1-api.md` §Task Creation.

---

### Schema Surface (this feature)

Creates one row in the `tasks` table on success. All columns written on creation:
- `id` (PK, auto-increment)
- `title`, `description`, `status`, `due_date`
- `created_at`, `updated_at` (both set to UTC now)

Full DDL: see `Y0-schema.md` §tasks table.
---

## F01: Task List View

**Description:** This feature renders the primary page of TaskFlow — a list of all tasks in the shared workspace. Team members land here by default and use it to get an at-a-glance overview of all work. The list supports status-based filtering (specified fully in F05) and links to individual task detail/edit views. It is the hub from which all other task operations are initiated.

---

### Terminology

- **Task Row:** A single task's summary entry in the list, showing title, status badge, due date (if set), and truncated description preview.
- **Status Badge:** A visually distinct UI element (chip, pill, or colored label) that displays the current status value of a task. Must meet WCAG AA color contrast.
- **Description Preview:** A truncated version of the task description shown inline in the list. Maximum 100 characters followed by ellipsis (`…`) if truncated.
- **Empty State:** The condition where the filtered or unfiltered task list contains zero tasks, requiring a specific informational message to be displayed.
- **Sort Order:** Default display order — newest tasks first, based on `created_at` descending.

---

### Sub-features

- Render the full task list (all tasks, no filter)
- Render filtered task list by status (see F05)
- Display each task as a summary row with key fields
- Display empty state when list is empty
- Link each task row to its detail/edit view
- Provide access to "New Task" creation action

---

### Process

1. User navigates to `GET /tasks` (with optional `?status=` query parameter — see F05 §Process).
2. Server reads the active status filter from the query parameter (if present and valid).
3. Server queries the `tasks` table: `SELECT * FROM tasks [WHERE status = ?] ORDER BY created_at DESC`.
4. If zero tasks match, server renders the empty state message (see §Empty State below).
5. If one or more tasks match, server renders each task as a task row.
6. Each task row displays: title (as a link to `GET /tasks/:id`), status badge, due date formatted as `MMM DD, YYYY` (e.g., `Jul 15, 2026`) or blank if NULL, description preview (first 100 chars + `…` if longer, or blank if NULL).
7. Filter controls are rendered above the list (see F05 §Process for filter rendering details).
8. A "New Task" button or link to `GET /tasks/new` is rendered prominently on the page.
9. Server returns HTTP 200 with the rendered page.

---

### Inputs

- `status` (query parameter, optional): Status filter value. Accepted URL values: `todo`, `in_progress`, `done`. Any other value is ignored and treated as no filter (show all). See F05 for full filter behavior.

---

### Outputs

**On success (tasks exist):**
- HTTP 200 with rendered HTML task list page.
- One task row per matching task, sorted `created_at DESC`.
- Filter controls rendered with the active filter visually indicated.

**On success (no tasks match):**
- HTTP 200 with rendered HTML task list page.
- Empty state message displayed: `"No tasks found."` (unfiltered) or `"No tasks with status '[Status Label]'."` (filtered).
- Filter controls still rendered.
- "New Task" button/link still rendered.

**On database error:**
- HTTP 500 error page with message: `"Unable to load tasks. Please try again."`

---

### Empty State Messages

| Condition | Message |
|-----------|---------|
| No tasks exist at all (unfiltered) | "No tasks yet. Create your first task to get started." |
| Filter active, no tasks match | "No tasks with status '[Status Label]'. Try a different filter or create a new task." |

`[Status Label]` is the human-readable label: `To Do`, `In Progress`, or `Done`.

---

### Validation Rules

- `status` query parameter: If provided, must be one of `todo`, `in_progress`, `done`. Invalid values are silently ignored (treated as no filter). No error is shown to the user.
- No other inputs require validation on this view.

---

### Error States

| Scenario | HTTP Status | Error Code | User-Facing Message |
|----------|-------------|------------|---------------------|
| Database read fails | 500 | `DB_READ_ERROR` | "Unable to load tasks. Please try again." |
| Invalid `status` query param | (ignored) | — | No error; fall back to unfiltered list |

---

### API Surface (this feature)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/tasks` | Render task list; optional `?status=` filter |

Full request/response schemas: see `Y1-api.md` §Task List.

---

### Schema Surface (this feature)

Reads from `tasks` table. No writes. Query pattern:
- Unfiltered: `SELECT id, title, description, status, due_date, created_at FROM tasks ORDER BY created_at DESC`
- Filtered: `SELECT id, title, description, status, due_date, created_at FROM tasks WHERE status = $1 ORDER BY created_at DESC`

Full DDL: see `Y0-schema.md` §tasks table.
---

## F02: Task Edit

**Description:** This feature allows any team member to modify any field of an existing task — title, description, status, or due date — and persist the changes to the database. The edit form is pre-populated with the task's current values so users can change only what they need. On successful save the user is returned to the task list where the updated values are immediately reflected. Validation rules are identical to those in F00 (Task Creation).

---

### Terminology

- **Edit Form:** The HTML form rendered at `GET /tasks/:id/edit` pre-populated with the task's current field values.
- **Task ID:** The integer primary key (`id`) of the task row being edited. Supplied as a URL path parameter.
- **Pre-population:** The server reads the existing task record and sets each form field's default value to the current stored value before rendering the form.
- **Partial Update:** Only the fields submitted in the form body are updated; `created_at` is never modified by an edit operation.

---

### Sub-features

- Render the edit form pre-populated with current task values (`GET /tasks/:id/edit`)
- Submit edited values (`PUT /tasks/:id` or `POST /tasks/:id` with method override)
- Server-side validation (identical rules to F00)
- Persist validated changes to the database
- Redirect to task list on success
- Re-render edit form with errors on validation failure
- Handle task not found (ID does not exist)

---

### Process

1. User clicks a task row or an "Edit" action link from the task list, navigating to `GET /tasks/:id/edit`.
2. Server looks up the task by `id` in the `tasks` table.
3. **If task not found:** Server returns HTTP 404 (see §Error States).
4. **If task found:** Server renders the edit form with all four fields pre-populated from the stored task record.
5. User modifies one or more fields.
6. User submits the form (`PUT /tasks/:id` or `POST /tasks/:id` with `_method=PUT` override for server-rendered HTML).
7. Server trims whitespace from all text inputs.
8. Server validates inputs using the same rules as F00 §Validation.
9. **If validation fails:** Server re-renders the edit form, pre-populated with the *submitted* (not original) values, with inline error messages. HTTP 422.
10. **If validation passes:** Server updates the matching `tasks` row: sets `title`, `description`, `status`, `due_date` to the submitted values, sets `updated_at` to current UTC timestamp. `created_at` is NOT modified.
11. Server responds with HTTP 302 redirect to `GET /tasks`.
12. Task list renders with the task's updated values visible.

---

### Inputs

- `:id` (path parameter, integer, required): The `id` of the task to edit. Must reference an existing row.
- `title` (VARCHAR(255), required): Updated task name. Same constraints as F00.
- `description` (TEXT, optional): Updated description. NULL if field is cleared/empty.
- `status` (ENUM, required): Updated status. Must be one of `To Do`, `In Progress`, `Done`. Cannot be blank on edit (unlike creation where it defaults silently).
- `due_date` (DATE `YYYY-MM-DD`, optional): Updated due date. NULL if field is cleared/empty.

---

### Outputs

**On successful save:**
- HTTP 302 redirect to `GET /tasks`.
- `tasks` row updated: `title`, `description`, `status`, `due_date`, `updated_at` reflect new values. `created_at` unchanged.

**On validation failure:**
- HTTP 422 with re-rendered edit form.
- Form fields populated with submitted (not original) values.
- Inline error messages displayed.

**On task not found:**
- HTTP 404 with error page.

---

### Validation Rules

- `title`: Same as F00 — non-empty after trim, ≤ 255 characters.
- `status`: Must be one of `To Do`, `In Progress`, `Done`. Cannot be empty (edit form always has a selected value in the dropdown).
- `due_date`: If provided and non-empty, must be a valid `YYYY-MM-DD` calendar date. Empty string treated as NULL (clear the due date).
- `:id`: Must be a positive integer. Must reference an existing `tasks` row.

---

### Error States

| Scenario | HTTP Status | Error Code | User-Facing Message |
|----------|-------------|------------|---------------------|
| Task not found (`:id` has no matching row) | 404 | `TASK_NOT_FOUND` | "Task not found." |
| Title is empty or whitespace | 422 | `TITLE_REQUIRED` | "Title is required." |
| Title exceeds 255 characters | 422 | `TITLE_TOO_LONG` | "Title must be 255 characters or fewer." |
| `due_date` is not a valid date | 422 | `DUE_DATE_INVALID` | "Due date must be a valid date (YYYY-MM-DD)." |
| `status` is missing or invalid | 422 | `STATUS_INVALID` | "Status must be one of: To Do, In Progress, Done." |
| Database write fails | 500 | `DB_WRITE_ERROR` | "Unable to save the task. Please try again." |
| `:id` is not a positive integer | 400 | `INVALID_TASK_ID` | "Invalid task ID." |

---

### API Surface (this feature)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/tasks/:id/edit` | Render edit form pre-populated with task values |
| `PUT` | `/tasks/:id` | Submit updated task fields; persist to DB |

Full request/response schemas: see `Y1-api.md` §Task Edit.

---

### Schema Surface (this feature)

Reads from and writes to `tasks` table.
- Read: `SELECT * FROM tasks WHERE id = $1`
- Update: `UPDATE tasks SET title=$1, description=$2, status=$3, due_date=$4, updated_at=NOW() WHERE id=$5`

`created_at` is never modified by this feature. Full DDL: see `Y0-schema.md` §tasks table.
---

## F03: Task Deletion

**Description:** This feature allows any team member to permanently delete a task from the shared workspace. Because deletion is irreversible, a mandatory confirmation step is interposed between the user's initial delete intent and the actual database removal. The confirmation can be implemented as a browser `confirm()` dialog, an inline JavaScript modal, or a dedicated server-rendered confirmation page — the exact UI mechanism is an implementation choice, but the confirmation step itself is non-negotiable. Once confirmed and deleted, the task is permanently removed and immediately disappears from the list.

---

### Terminology

- **Delete Intent:** The user's first action indicating they want to delete a task (clicking a "Delete" button or link).
- **Confirmation Step:** The mandatory intermediate step that presents the question `"Are you sure you want to delete this task?"` and requires an affirmative response before deletion proceeds.
- **Permanent Deletion:** A hard delete — the row is removed from the `tasks` table. Soft-delete is a v1.1 candidate but out of scope for v1.
- **Cancellation:** The user declines the confirmation, no database change occurs, user is returned to their prior view.

---

### Sub-features

- Render delete action (button/link) on task list and/or task detail page
- Present confirmation step before deletion
- Execute permanent deletion on confirmation (`DELETE /tasks/:id`)
- Redirect to task list on confirmed deletion
- Handle cancellation (no change, return to list)
- Handle task not found

---

### Process

1. User clicks a "Delete" button or link associated with a specific task (available from the task list row and/or task detail view).
2. System presents the confirmation step: `"Are you sure you want to delete '[task title]'? This action cannot be undone."`
3. **If user cancels:** No request is sent (or a cancel action redirects to `GET /tasks`). Task remains in the database. User returns to the task list.
4. **If user confirms:** Client sends `DELETE /tasks/:id` (or `POST /tasks/:id/delete` with method override for HTML forms).
5. Server looks up the task by `id` in the `tasks` table.
6. **If task not found:** Server returns HTTP 404 (task may have already been deleted by another team member).
7. **If task found:** Server executes `DELETE FROM tasks WHERE id = $1`.
8. Server responds with HTTP 302 redirect to `GET /tasks`.
9. Task list renders without the deleted task.

---

### Inputs

- `:id` (path parameter, integer, required): The `id` of the task to delete. Must reference an existing row.
- Confirmation user gesture (UI-level, not HTTP parameter): Affirmative response to the confirmation dialog/page.

---

### Outputs

**On confirmed deletion:**
- HTTP 302 redirect to `GET /tasks`.
- Task row permanently removed from `tasks` table.
- Task no longer appears in the task list for any viewer.

**On cancellation:**
- No HTTP request to the delete endpoint is made.
- User remains on or is redirected to `GET /tasks`.
- No database change.

**On task not found:**
- HTTP 404 with error page.

---

### Validation Rules

- `:id`: Must be a positive integer. Must reference an existing `tasks` row at the time of the delete request.
- No other field inputs exist for this operation.
- Confirmation step is a UX gate, not a form validation — it must be present in the UI but is not enforced at the HTTP layer (the API accepts a DELETE request directly).

---

### Error States

| Scenario | HTTP Status | Error Code | User-Facing Message |
|----------|-------------|------------|---------------------|
| Task not found (`:id` has no matching row) | 404 | `TASK_NOT_FOUND` | "Task not found. It may have already been deleted." |
| `:id` is not a positive integer | 400 | `INVALID_TASK_ID` | "Invalid task ID." |
| Database delete fails | 500 | `DB_DELETE_ERROR` | "Unable to delete the task. Please try again." |

---

### API Surface (this feature)

| Method | Path | Description |
|--------|------|-------------|
| `DELETE` | `/tasks/:id` | Permanently delete task by ID |
| `POST` | `/tasks/:id/delete` | HTML-form method override for DELETE |

Full request/response schemas: see `Y1-api.md` §Task Deletion.

---

### Schema Surface (this feature)

Writes to `tasks` table (hard delete).
- Lookup: `SELECT id FROM tasks WHERE id = $1`
- Delete: `DELETE FROM tasks WHERE id = $1`

No soft-delete column exists in the v1 schema. Full DDL: see `Y0-schema.md` §tasks table.
---

## F04: Data Persistence

**Description:** This feature defines the infrastructure requirement that all task data is stored in a relational database and survives application server restarts. It is not a user-facing feature but an architectural constraint that underpins the correctness of all other features. In-memory or session-scoped storage is explicitly prohibited. Every create, update, and delete operation must write synchronously to the database, and the HTTP response must not be sent until the database write has been confirmed. The schema must be reproducible from scratch via migration scripts or an ORM migration command.

---

### Terminology

- **Relational Database:** A structured data store using tables, rows, and columns, supporting SQL queries and ACID transactions. PostgreSQL is the production default; SQLite is permitted for local development and testing only.
- **In-Memory Store:** Any data structure (JavaScript object, Python dict, array, etc.) that holds task data in application process memory. Explicitly prohibited for task records in v1.
- **Migration:** A versioned, idempotent script that creates or modifies the database schema. Must be runnable from the command line to set up a fresh database.
- **Synchronous Write:** A database write operation that completes (and is acknowledged by the DB) before the HTTP response is sent to the client. The server must not respond with a success redirect before the INSERT/UPDATE/DELETE has been committed.
- **Schema Drift:** A condition where the live database schema diverges from the version-controlled migration scripts. Prevented by using ORM migrations from project inception.

---

### Sub-features

- Define and version-control the `tasks` table schema
- Provide a migration command to create the schema from scratch
- Ensure all writes (create, update, delete) are synchronous and confirmed before response
- Ensure task list reads directly from the database on every request (no caching layer)
- Support PostgreSQL (production) and SQLite (local dev/test) without code changes
- Survive server restarts without data loss

---

### Process

1. **Initial setup:** Developer runs the migration command (e.g., `npx sequelize-cli db:migrate` or `python manage.py migrate`) against a new or existing database. The `tasks` table is created if it does not exist.
2. **Runtime — write operations:** On every task create, update, or delete:
   a. Application issues the appropriate SQL statement via ORM.
   b. Application awaits ORM promise/async resolution (confirming DB write).
   c. Only after confirmed write does the application issue the HTTP redirect or response.
3. **Runtime — read operations:** On every task list or task detail request, the application queries the database directly. No in-memory cache, no session storage.
4. **Server restart:** Application process stops and restarts. No task data is lost because all data resides in the database. The application reconnects to the database on startup.
5. **Schema changes (future):** New migrations are added and run. Rollback scripts are provided for each migration.

---

### Inputs

- No direct user inputs for this feature.
- Indirectly, all write operations from F00, F02, F03 feed into this persistence layer.
- Database connection parameters (DSN/URL) provided via environment variable (e.g., `DATABASE_URL`).

---

### Outputs

- `tasks` table exists in the database after migration is run.
- All task data written by F00/F02/F03 is durable across server restarts.
- Database connection error surfaces as HTTP 500 to users (see §Error States).

---

### Validation Rules

- **No in-memory storage:** Any implementation using a module-level array, object, or Map to hold task records between requests is non-compliant.
- **Synchronous writes:** Any implementation that sends an HTTP 2xx/3xx before `await db.save()` (or ORM equivalent) resolves is non-compliant.
- **Migration required:** Schema must be created via a migration tool, not via a one-off `CREATE TABLE` command run manually. The migration file must be version-controlled.
- **SQLite restriction:** SQLite may be used for local development and testing. PostgreSQL must be used for any shared or production deployment. The application must support switching between the two via environment variable only (no code changes required).
- **No raw SQL:** All queries must use ORM methods or parameterized queries. Raw string-concatenated SQL is non-compliant.

---

### Error States

| Scenario | HTTP Status | Error Code | User-Facing Message |
|----------|-------------|------------|---------------------|
| Database connection fails on startup | N/A (process crash or health check failure) | `DB_CONNECTION_ERROR` | "Unable to connect to the database." (logged; health endpoint returns 503) |
| Database write fails mid-request | 500 | `DB_WRITE_ERROR` | "Unable to save the task. Please try again." |
| Database read fails mid-request | 500 | `DB_READ_ERROR` | "Unable to load tasks. Please try again." |
| Migration not run (table missing) | 500 | `DB_SCHEMA_MISSING` | "Database schema not initialized. Run migrations." (logged; not shown in prod) |

---

### API Surface (this feature)

This feature has no dedicated API endpoints. It is a cross-cutting infrastructure concern enforced by the implementation of F00, F01, F02, and F03.

---

### Schema Surface (this feature)

Defines the canonical `tasks` table schema. Full DDL: see `Y0-schema.md` §tasks table.

Key column constraints:
- `id`: SERIAL / AUTOINCREMENT primary key. Never NULL.
- `title`: VARCHAR(255) NOT NULL. Enforced at DB level in addition to application validation.
- `status`: VARCHAR(20) NOT NULL DEFAULT 'To Do'. CHECK constraint: `status IN ('To Do', 'In Progress', 'Done')`.
- `created_at` / `updated_at`: TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(). `created_at` never updated after insert; `updated_at` always updated on each UPDATE.
---

## F05: Status Filtering

**Description:** This feature allows team members to narrow the task list to only tasks matching a specific status value, reducing visual noise when the team wants to focus on a particular category of work. Filter controls are rendered on the task list page, the active filter is visually indicated, and the URL encodes the active filter so filtered views can be bookmarked or shared. The filter state persists within a browser session via URL (no server-side session required). This feature is implemented as part of the task list view (F01) but specified separately due to its distinct UX and URL-routing behavior.

---

### Terminology

- **Filter Control:** The UI element (tabs, buttons, or dropdown) that allows the user to select a status filter. Rendered above the task list on the task list page.
- **Active Filter:** The currently selected filter value. Visually distinguished from inactive filters (e.g., bold text, underline, filled button style).
- **Status Slug:** The URL-safe filter value used in the `?status=` query parameter. Maps to display labels as follows:

| Slug | Display Label |
|------|---------------|
| `todo` | To Do |
| `in_progress` | In Progress |
| `done` | Done |
| (absent or `all`) | All |

- **Bookmarkable Filter:** A filtered task list URL (e.g., `/tasks?status=in_progress`) that, when shared or bookmarked, returns the same filtered view on reload.

---

### Sub-features

- Render filter controls on the task list page
- Accept `?status=` query parameter and apply to DB query
- Visually indicate the active filter
- Encode active filter in the URL
- Handle invalid/unknown filter values gracefully (fall back to "All")
- Display appropriate empty state when no tasks match the active filter (see F01 §Empty State Messages)

---

### Process

1. User is on the task list page (`GET /tasks`). Filter controls are always visible regardless of active filter.
2. Filter controls render four options: `All`, `To Do`, `In Progress`, `Done`.
3. Each filter option is a link to `GET /tasks?status={slug}` (or `GET /tasks` for "All").
4. The option matching the current `?status=` parameter value is styled as active.
5. If no `?status=` parameter is present, the `All` option is active.
6. If `?status=` is present but not a recognized slug, the `All` filter is applied silently (no error).
7. User clicks a filter option — browser navigates to the corresponding URL.
8. Server receives `GET /tasks?status={slug}`, reads the slug, maps it to the DB enum value (e.g., `in_progress` → `In Progress`), and queries: `SELECT * FROM tasks WHERE status = $1 ORDER BY created_at DESC`.
9. For "All" (no `?status=` or `?status=all`): query has no `WHERE status` clause.
10. Results rendered as task rows (see F01 §Process steps 5–6).
11. If zero results match the active filter, empty state message is rendered (see F01 §Empty State Messages).

---

### Inputs

- `status` (query parameter, optional string): One of `todo`, `in_progress`, `done`, `all`. Any other value treated as `all`.

---

### Outputs

- HTTP 200 with task list page.
- Task rows filtered to match the active status (or all tasks if no filter / `all`).
- Filter controls rendered with the active option visually indicated.
- URL contains `?status={slug}` reflecting the active filter (for bookmarking/sharing).

---

### Validation Rules

- `status` query param: Accepted values are `todo`, `in_progress`, `done`, `all`, and absent. All other values silently treated as no filter (show all). No validation error is shown.
- Filter controls must always be rendered even when the list is empty, so users can switch to a different filter.
- The active filter indicator must be driven by the URL parameter, not client-side state, so the page is stateless and refresh-safe.

---

### Slug-to-Enum Mapping

| Query Param `?status=` | DB `status` value queried | Filter label shown as active |
|------------------------|---------------------------|------------------------------|
| (absent) | (no WHERE clause) | `All` |
| `all` | (no WHERE clause) | `All` |
| `todo` | `To Do` | `To Do` |
| `in_progress` | `In Progress` | `In Progress` |
| `done` | `Done` | `Done` |
| (any other value) | (no WHERE clause) | `All` |

---

### Error States

| Scenario | HTTP Status | Error Code | User-Facing Message |
|----------|-------------|------------|---------------------|
| Unknown `status` slug | (no error) | — | Silently falls back to unfiltered list; `All` shown as active |
| Database read fails with filter applied | 500 | `DB_READ_ERROR` | "Unable to load tasks. Please try again." |

---

### API Surface (this feature)

This feature uses the same endpoint as F01. The `?status=` query parameter extends it:

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/tasks?status={slug}` | Task list filtered by status slug |

Full request/response schemas: see `Y1-api.md` §Task List (Status Filter).

---

### Schema Surface (this feature)

Reads from `tasks` table with an optional `WHERE status = $1` clause. No schema additions beyond what F04 defines. Full DDL: see `Y0-schema.md` §tasks table.

The `status` column has a DB-level CHECK constraint ensuring only valid values exist, which means any data returned from the DB is always a valid enum value.
---

## Y0: Database Schema

**Scope:** Full DDL for all TaskFlow v1 entities. TaskFlow v1 has a single domain table: `tasks`. No user, session, or tenant tables exist (no authentication, no multi-tenancy).

**Database Support:**
- **Production:** PostgreSQL 14+
- **Development/Testing:** SQLite 3.x (with ORM-managed dialect differences)

The DDL below is written in PostgreSQL syntax. ORM migration tools (Sequelize / Django ORM) generate the equivalent SQLite DDL automatically when the `DATABASE_URL` points to a SQLite file.

---

### tasks Table

```sql
-- PostgreSQL DDL

CREATE TABLE tasks (
    id          SERIAL PRIMARY KEY,
    title       VARCHAR(255)  NOT NULL,
    description TEXT          NULL,
    status      VARCHAR(20)   NOT NULL DEFAULT 'To Do'
                    CHECK (status IN ('To Do', 'In Progress', 'Done')),
    due_date    DATE          NULL,
    created_at  TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

---

### Column Definitions

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| `id` | SERIAL (PostgreSQL) / INTEGER AUTOINCREMENT (SQLite) | NOT NULL | auto | Primary Key | System-assigned; never set by application code |
| `title` | VARCHAR(255) | NOT NULL | — | ≤ 255 chars | Also validated at application layer (F00, F02) |
| `description` | TEXT | NULL | NULL | — | Stored as NULL when not provided; no length limit at DB layer |
| `status` | VARCHAR(20) | NOT NULL | `'To Do'` | CHECK: one of `'To Do'`, `'In Progress'`, `'Done'` | DB-level constraint mirrors application-level enum |
| `due_date` | DATE | NULL | NULL | Valid calendar date | No time component; NULL when not provided |
| `created_at` | TIMESTAMP WITH TIME ZONE | NOT NULL | `NOW()` | — | Set at INSERT; never updated |
| `updated_at` | TIMESTAMP WITH TIME ZONE | NOT NULL | `NOW()` | — | Set at INSERT; updated on every UPDATE |

---

### Indexes

```sql
-- Default index on primary key (created automatically)
-- Additional indexes for query performance

CREATE INDEX idx_tasks_status ON tasks (status);
CREATE INDEX idx_tasks_created_at ON tasks (created_at DESC);
```

- `idx_tasks_status`: Supports `WHERE status = $1` queries used by F01/F05 status filtering.
- `idx_tasks_created_at`: Supports `ORDER BY created_at DESC` default sort on the task list.

---

### Migration Script

The schema must be created and managed via ORM migration tools, not via manually executed SQL. The above DDL represents the target schema state after the initial migration runs.

**Node.js / Sequelize:**
```bash
npx sequelize-cli db:migrate
```

**Python / Django:**
```bash
python manage.py migrate
```

**Environment Variable:**
```
DATABASE_URL=postgres://user:password@host:5432/taskflow   # PostgreSQL (production)
DATABASE_URL=sqlite:///./taskflow.db                        # SQLite (local dev)
```

---

### Data Constraints Summary

- A `tasks` row with `title = NULL` must never be inserted (enforced at both application and DB layers).
- A `tasks` row with `status` outside the allowed enum must never be inserted (enforced at both application and DB layers).
- `id` values are never reused after deletion (SERIAL/AUTOINCREMENT).
- No foreign keys exist in v1 (single-table design).
- No soft-delete column (`deleted_at`, `is_deleted`) exists in v1 — all deletes are hard deletes.

---

### Future Schema Notes (out of scope for v1)

The following columns/tables are candidates for v2+ but must NOT be added in v1:
- `assigned_to` (FK to a users table) — requires authentication
- `deleted_at` (soft-delete) — v1.1 candidate
- `project_id` / `workspace_id` — multi-tenancy, v2+
- `comments` table — v2+
---

## Y1: REST API Endpoint Catalog

**Scope:** Complete specification of all HTTP endpoints exposed by the TaskFlow v1 server. All endpoints return server-rendered HTML (not JSON) in the default implementation, but the request/response contracts are specified here for both HTML and JSON representations to support future API clients.

**Base URL:** `http://{host}:{port}` (no path prefix in v1; configurable via environment).

**Content-Type:** Server-rendered HTML responses use `text/html`. If a JSON API layer is added, `application/json` is used. This document specifies both.

**No Authentication:** No `Authorization` header, no session cookie required. All endpoints are accessible to any client.

---

### Endpoint Summary

| # | Method | Path | Feature | Description |
|---|--------|------|---------|-------------|
| 1 | `GET` | `/tasks` | F01, F05 | Task list view (optionally filtered) |
| 2 | `GET` | `/tasks/new` | F00 | Task creation form |
| 3 | `POST` | `/tasks` | F00 | Create new task |
| 4 | `GET` | `/tasks/:id` | F01, F02 | Task detail view (read-only) |
| 5 | `GET` | `/tasks/:id/edit` | F02 | Task edit form |
| 6 | `PUT` | `/tasks/:id` | F02 | Update task |
| 7 | `POST` | `/tasks/:id` | F02 | Update task (HTML method override via `_method=PUT`) |
| 8 | `DELETE` | `/tasks/:id` | F03 | Delete task |
| 9 | `POST` | `/tasks/:id/delete` | F03 | Delete task (HTML method override) |
| 10 | `GET` | `/` | — | Root redirect to `GET /tasks` |

---

### §Task List

#### `GET /tasks`

**Description:** Returns the task list, optionally filtered by status.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Filter by status slug: `todo`, `in_progress`, `done`, `all`. Unknown values ignored (all tasks returned). |

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 200 | Success (tasks found or empty) | HTML task list page |
| 500 | Database error | HTML error page |

**Example URLs:**
- `GET /tasks` — all tasks
- `GET /tasks?status=todo` — only "To Do" tasks
- `GET /tasks?status=in_progress` — only "In Progress" tasks
- `GET /tasks?status=done` — only "Done" tasks

---

### §Task Creation

#### `GET /tasks/new`

**Description:** Returns the task creation form.

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 200 | Success | HTML creation form with empty fields; status defaulted to `To Do` |

---

#### `POST /tasks`

**Description:** Creates a new task.

**Request Body (form-encoded):**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | Yes | Non-empty after trim; ≤ 255 chars |
| `description` | string | No | Any text; defaults to NULL |
| `status` | string | No | One of `To Do`, `In Progress`, `Done`; defaults to `To Do` |
| `due_date` | string (YYYY-MM-DD) | No | Valid calendar date or empty; defaults to NULL |

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 302 | Task created successfully | Redirect to `GET /tasks` |
| 422 | Validation failure | HTML creation form with error messages and submitted values pre-populated |
| 500 | Database write error | HTML error page |

---

### §Task Detail

#### `GET /tasks/:id`

**Description:** Returns the detail view for a single task (read-only).

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | Yes | Task primary key |

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 200 | Task found | HTML task detail page |
| 404 | Task not found | HTML 404 error page |
| 400 | `:id` not a positive integer | HTML 400 error page |

---

### §Task Edit

#### `GET /tasks/:id/edit`

**Description:** Returns the task edit form pre-populated with current values.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | Yes | Task primary key |

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 200 | Task found | HTML edit form with fields populated |
| 404 | Task not found | HTML 404 error page |
| 400 | `:id` not a positive integer | HTML 400 error page |

---

#### `PUT /tasks/:id` (also `POST /tasks/:id` with `_method=PUT`)

**Description:** Updates an existing task.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | Yes | Task primary key |

**Request Body (form-encoded):**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | Yes | Non-empty after trim; ≤ 255 chars |
| `description` | string | No | Any text; empty string treated as NULL |
| `status` | string | Yes | One of `To Do`, `In Progress`, `Done` |
| `due_date` | string (YYYY-MM-DD) | No | Valid calendar date or empty (NULL) |
| `_method` | string | No | Set to `PUT` for HTML form method override |

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 302 | Task updated successfully | Redirect to `GET /tasks` |
| 404 | Task not found | HTML 404 error page |
| 422 | Validation failure | HTML edit form with error messages |
| 400 | `:id` not a positive integer | HTML 400 error page |
| 500 | Database write error | HTML error page |

---

### §Task Deletion

#### `DELETE /tasks/:id` (also `POST /tasks/:id/delete`)

**Description:** Permanently deletes a task.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | Yes | Task primary key |

**Request Body (for `POST /tasks/:id/delete` form override):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `_method` | string | No | Set to `DELETE` |

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 302 | Task deleted successfully | Redirect to `GET /tasks` |
| 404 | Task not found | HTML 404 error page |
| 400 | `:id` not a positive integer | HTML 400 error page |
| 500 | Database delete error | HTML error page |

---

### §Root Redirect

#### `GET /`

**Description:** Root URL redirects to the task list.

**Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 302 | Always | Redirect to `GET /tasks` |
---

## Y2: Cross-Feature Error Catalog

**Scope:** Comprehensive catalog of all error scenarios across TaskFlow v1, organized by HTTP status code. Each entry identifies the triggering feature(s), the internal error code, the user-facing message, and guidance on the appropriate client/server behavior.

**Design Principle:** Errors are surfaced as server-rendered HTML pages or inline form messages. JSON error responses are not part of the v1 spec but the error codes are defined here for future API layer compatibility.

---

### 400 Bad Request

These errors occur when the request itself is malformed (before any business logic or DB access).

| Error Code | Feature(s) | Trigger | User-Facing Message | Server Behavior |
|------------|------------|---------|---------------------|-----------------|
| `INVALID_TASK_ID` | F02, F03 | `:id` path parameter is not a positive integer (e.g., `/tasks/abc`, `/tasks/-1`, `/tasks/0`) | "Invalid task ID." | Return 400 HTML error page; do not query DB |

---

### 404 Not Found

These errors occur when a referenced resource does not exist in the database.

| Error Code | Feature(s) | Trigger | User-Facing Message | Server Behavior |
|------------|------------|---------|---------------------|-----------------|
| `TASK_NOT_FOUND` | F02, F03 | Task `:id` has no matching row in `tasks` table | "Task not found." (F02: "Task not found.") / "Task not found. It may have already been deleted." (F03) | Return 404 HTML error page; do not attempt write operation |

---

### 422 Unprocessable Entity

These errors occur when the request is syntactically valid but fails business-logic validation.

| Error Code | Feature(s) | Trigger | User-Facing Message | Server Behavior |
|------------|------------|---------|---------------------|-----------------|
| `TITLE_REQUIRED` | F00, F02 | `title` is empty, null, or whitespace-only after trimming | "Title is required." | Re-render form with error; pre-populate submitted values |
| `TITLE_TOO_LONG` | F00, F02 | `title` exceeds 255 characters after trimming | "Title must be 255 characters or fewer." | Re-render form with error; pre-populate submitted values |
| `DUE_DATE_INVALID` | F00, F02 | `due_date` is provided but not a valid `YYYY-MM-DD` calendar date | "Due date must be a valid date (YYYY-MM-DD)." | Re-render form with error; pre-populate submitted values |
| `STATUS_INVALID` | F02 | `status` submitted value is not one of `To Do`, `In Progress`, `Done` (only on edit, where status is required) | "Status must be one of: To Do, In Progress, Done." | Re-render form with error |

**Note on multiple errors:** If multiple fields are invalid simultaneously, all applicable error messages must be displayed together on the re-rendered form (not just the first error).

---

### 500 Internal Server Error

These errors occur when the application encounters an unexpected failure, typically a database error.

| Error Code | Feature(s) | Trigger | User-Facing Message | Server Behavior |
|------------|------------|---------|---------------------|-----------------|
| `DB_READ_ERROR` | F01, F05 | Database query fails when fetching task list or single task | "Unable to load tasks. Please try again." | Return 500 HTML error page; log full error server-side |
| `DB_WRITE_ERROR` | F00, F02 | Database INSERT or UPDATE fails after validation passes | "Unable to save the task. Please try again." | Return 500 HTML error page; log full error server-side; do NOT redirect |
| `DB_DELETE_ERROR` | F03 | Database DELETE fails | "Unable to delete the task. Please try again." | Return 500 HTML error page; log full error server-side |
| `DB_CONNECTION_ERROR` | F04 (all) | Database connection unavailable at request time | "The application is temporarily unavailable. Please try again later." | Return 503 HTML error page; health check endpoint returns 503 |
| `DB_SCHEMA_MISSING` | F04 | Table `tasks` does not exist (migration not run) | "Database schema not initialized. Contact the administrator." | Return 500 HTML error page; log clearly that migration has not been run |

---

### Silently Handled Conditions (No Error Shown)

These conditions are handled gracefully without surfacing an error to the user:

| Condition | Feature(s) | Behavior |
|-----------|------------|----------|
| Unknown `?status=` query parameter value | F01, F05 | Silently treated as no filter; "All" tab shown as active |
| `status` field absent or invalid on task creation | F00 | Silently defaulted to `To Do`; no error shown |
| Empty `due_date` field on edit (user clears the date) | F02 | Treated as NULL (due date removed); no error shown |
| Empty `description` field | F00, F02 | Treated as NULL; no error shown |

---

### Error Display Requirements

- **Inline form errors (422):** Error messages must appear adjacent to the invalid field (not only at the top of the page). A summary at the top of the form is permitted in addition to inline messages.
- **Error pages (400, 404, 500, 503):** Must include a link back to `GET /tasks` so the user can recover without using the browser back button.
- **Server-side logging:** All 500-level errors must be logged with full stack trace on the server. User-facing messages must NOT include stack traces, database error details, or internal identifiers.
- **Security:** Error pages must not reveal database structure, file paths, or environment details to the user.
---

## Y3: External Integration Points

**Scope:** All external systems and dependencies that TaskFlow v1 interacts with at runtime or development time. TaskFlow v1 is intentionally minimal and has very few external integration points.

---

### Integration Summary

| # | System | Type | Required | Scope |
|---|--------|------|----------|-------|
| 1 | Relational Database (PostgreSQL / SQLite) | Data store | Yes | All features (runtime) |
| 2 | Node.js / Python Runtime | Execution environment | Yes | All features (runtime) |
| 3 | ORM (Sequelize / Django ORM) | Query abstraction layer | Yes | All features (development + runtime) |
| 4 | Browser (client-side rendering) | User interface | Yes | All features (client) |

There are **no third-party external service calls** in TaskFlow v1. No email, no push notifications, no external authentication providers, no analytics services, no CDNs, no object storage.

---

### Integration 1: Relational Database

**System:** PostgreSQL 14+ (production) / SQLite 3.x (development)
**Purpose:** Persistent storage of all task data
**Protocol:** TCP (PostgreSQL) / local file (SQLite)
**Configuration:** Via `DATABASE_URL` environment variable

| Property | Value |
|----------|-------|
| Connection method | ORM connection pool (Sequelize) or Django DB settings |
| Connection string format | `postgres://user:pass@host:port/dbname` or `sqlite:///path/to/file.db` |
| Max connections (PostgreSQL) | 5–10 (suitable for small team, single server) |
| Auth mechanism | Username + password (PostgreSQL); none (SQLite) |
| SSL | Recommended for production PostgreSQL; not required for SQLite |
| Failure behavior | HTTP 503 / 500; full error logged server-side |

**Dependency:** The application process cannot start successfully if `DATABASE_URL` is not set and valid. A startup health check should verify DB connectivity before accepting requests.

---

### Integration 2: Node.js / Python Runtime

**System:** Node.js 18+ (if Express) or Python 3.10+ (if Django)
**Purpose:** Application execution environment
**Type:** Process runtime — not an external service

| Property | Value |
|----------|-------|
| Node.js version | 18 LTS or higher |
| Python version | 3.10 or higher |
| Package manager | npm (Node) / pip + requirements.txt or poetry (Python) |
| Startup command | `npm start` (Node) / `python manage.py runserver` (Django) |
| Port | Configurable via `PORT` environment variable; default 3000 (Node) or 8000 (Django) |

---

### Integration 3: ORM (Sequelize / Django ORM)

**System:** Sequelize 6+ (Node) or Django ORM (built-in with Django 4+)
**Purpose:** Parameterized query generation, connection pooling, migration management
**Type:** Library dependency — not an external service

| Property | Value |
|----------|-------|
| Version | Sequelize 6.x (Node) / Django 4.x built-in ORM |
| Migration tool | sequelize-cli (Node) / `python manage.py migrate` (Django) |
| Dialect support | postgres, sqlite (both ORMs) |
| Raw SQL | Not used; all queries via ORM methods |
| Security benefit | Parameterized queries prevent SQL injection by default |

---

### Integration 4: Browser (Client-side)

**System:** Web browser (Chrome, Firefox, Safari, Edge — current versions)
**Purpose:** Renders server-generated HTML; handles form submissions
**Type:** Client — not a service integration

| Property | Value |
|----------|-------|
| Rendering model | Server-rendered HTML (no SPA framework required) |
| JavaScript requirement | Minimal; confirmation dialogs may use `window.confirm()` or a lightweight JS modal |
| CSS dependencies | None mandated; standard CSS is sufficient |
| Form method support | HTML forms only support GET/POST; PUT/DELETE require `_method` override in form body |
| Cookies / Sessions | Not used in v1 (no authentication, no server-side sessions) |

---

### Out-of-Scope Integrations (v2+)

The following integrations are explicitly NOT part of v1 and must not be implemented:

- **Email / Notifications:** No email delivery, no webhook, no Slack/Teams integration.
- **Authentication Provider:** No OAuth, SAML, LDAP, or SSO.
- **Object Storage:** No S3, GCS, or local file storage for attachments.
- **Analytics / Monitoring:** No Segment, Mixpanel, Sentry, Datadog, etc. (basic server logging is acceptable).
- **CDN:** No CDN for static assets (single-server deployment serves static files directly).
- **WebSockets / Server-Sent Events:** No real-time push; clients refresh the page to see updates.

---

*End of integration points document.*
