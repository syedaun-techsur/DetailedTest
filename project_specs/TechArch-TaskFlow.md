# Technical Architecture Document — TaskFlow

**Project Acronym:** TaskFlow  
**Version:** 1.0  
**Date:** 2026-07-01  
**Status:** Draft  
**Based On:** PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, PROJECT.md v1.0

---

## Table of Contents

1. [Architectural Overview](#1-architectural-overview)
2. [Component Architecture](#2-component-architecture)
3. [Data Model](#3-data-model)
4. [API Design](#4-api-design)
5. [Security Architecture](#5-security-architecture)
6. [Technology Stack](#6-technology-stack)
7. [Integration Points](#7-integration-points)

---

## 1. Architectural Overview

### 1.1 Architecture Pattern

TaskFlow uses a **Monolithic Server-Rendered Web Application** pattern — the simplest architecture that satisfies all requirements:

- A single Node.js/Express process handles HTTP routing, business logic, and HTML template rendering
- A single relational database (PostgreSQL in production, SQLite in development) provides durable CRUD storage
- No client-side SPA framework, no build pipeline, no microservices, no message queues
- All state lives in the database; the application server is stateless between requests

This pattern was chosen deliberately:
- **Zero authentication overhead** — no session store, no JWT service, no OAuth provider needed
- **Instant comprehensibility** — any developer can read the full codebase in under an hour (PRD success metric)
- **Single-command startup** — `npm start` with a valid `DATABASE_URL` is all that is needed
- **Conventional and maintainable** — Express + Sequelize (or Django + Django ORM) are the industry standard for this problem class

### 1.2 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Browser (Client)                           │
│                                                                     │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌────────────────┐  │
│  │ Task List │  │ New Task  │  │ Edit Task │  │  Delete Confirm │  │
│  │  Page     │  │  Form     │  │  Form     │  │    Dialog       │  │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └───────┬────────┘  │
└────────┼──────────────┼──────────────┼─────────────────┼───────────┘
         │ GET/POST/PUT/DELETE (HTTP/1.1)                 │
         ▼                                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Application Server (Express / Django)            │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                         Router Layer                          │  │
│  │  GET /          GET /tasks        GET /tasks/new              │  │
│  │  GET /tasks/:id  GET /tasks/:id/edit                          │  │
│  │  POST /tasks    PUT /tasks/:id    DELETE /tasks/:id           │  │
│  │  POST /tasks/:id  POST /tasks/:id/delete                      │  │
│  └────────────────────────────┬──────────────────────────────────┘  │
│                               │                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                      Controller Layer                         │  │
│  │  TaskController                                               │  │
│  │  ├── list()      → query + render task list                   │  │
│  │  ├── show()      → query single + render detail               │  │
│  │  ├── new()       → render creation form                       │  │
│  │  ├── create()    → validate + insert + redirect               │  │
│  │  ├── edit()      → query single + render edit form            │  │
│  │  ├── update()    → validate + update + redirect               │  │
│  │  └── destroy()   → lookup + delete + redirect                 │  │
│  └────────────────────────────┬──────────────────────────────────┘  │
│                               │                                     │
│  ┌────────────────────────────┴──────────────────────────────────┐  │
│  │                    Validation Layer                           │  │
│  │  validateTaskInput() → TITLE_REQUIRED, TITLE_TOO_LONG,        │  │
│  │                         DUE_DATE_INVALID, STATUS_INVALID      │  │
│  └────────────────────────────┬──────────────────────────────────┘  │
│                               │                                     │
│  ┌────────────────────────────┴──────────────────────────────────┐  │
│  │                    ORM / Query Layer                          │  │
│  │  Sequelize 6.x (Node) / Django ORM (Python)                   │  │
│  │  Parameterized queries only — no raw SQL                      │  │
│  └────────────────────────────┬──────────────────────────────────┘  │
│                               │                                     │
│  ┌────────────────────────────┴──────────────────────────────────┐  │
│  │                    Template / View Layer                      │  │
│  │  Server-rendered HTML templates (EJS / Jinja2 / Handlebars)   │  │
│  │  Output-escaped by default — XSS prevention                   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
└────────────────────────────────┬────────────────────────────────────┘
                                 │ TCP (PostgreSQL) / File I/O (SQLite)
                                 │ via DATABASE_URL env var
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         Database Layer                              │
│                                                                     │
│  PostgreSQL 14+ (production)    SQLite 3.x (development/testing)    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  tasks table                                                │    │
│  │  id | title | description | status | due_date |            │    │
│  │  created_at | updated_at                                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.3 Deployment Topology

```
┌──────────────────────────────────────────────┐
│               Single Server / Container       │
│                                              │
│  ┌───────────────────────────────────────┐   │
│  │         Application Process           │   │
│  │   Node.js 18+ / Python 3.10+          │   │
│  │   Port: 3000 (Node) / 8000 (Django)   │   │
│  │   Configurable via PORT env var       │   │
│  └───────────────────┬───────────────────┘   │
│                      │ localhost connection   │
│  ┌───────────────────┴───────────────────┐   │
│  │         PostgreSQL 14+                │   │
│  │   Connection pool: 5–10 connections   │   │
│  │   Configured via DATABASE_URL         │   │
│  └───────────────────────────────────────┘   │
│                                              │
└──────────────────────────────────────────────┘
```

**Key deployment decisions:**
- Single process, single server — no orchestration (Kubernetes, ECS) required for a ≤15 person internal tool
- PostgreSQL co-located or on a separate managed instance (e.g., RDS, Supabase, Railway) — both are valid
- No reverse proxy required for development; optional Nginx/Caddy for production TLS termination
- Container-ready (Dockerfile with `CMD ["npm", "start"]`) but not required — bare Node.js on a VM is also valid

---

## 2. Component Architecture

### 2.1 Backend Components

#### Router (`src/routes/tasks.js` or `tasks/urls.py`)

**Responsibility:** Map incoming HTTP method + path combinations to controller actions. Handle HTML form method overrides (`_method=PUT`, `_method=DELETE`).

**Routes registered:**

| Route | Controller Action |
|-------|-------------------|
| `GET /` | Redirect to `/tasks` |
| `GET /tasks` | `TaskController.list` |
| `GET /tasks/new` | `TaskController.new` |
| `POST /tasks` | `TaskController.create` |
| `GET /tasks/:id` | `TaskController.show` |
| `GET /tasks/:id/edit` | `TaskController.edit` |
| `PUT /tasks/:id` | `TaskController.update` |
| `POST /tasks/:id` | `TaskController.update` (method override) |
| `DELETE /tasks/:id` | `TaskController.destroy` |
| `POST /tasks/:id/delete` | `TaskController.destroy` (method override) |

**Implementation note:** For Express, use `method-override` middleware to support `_method` in POST body. For Django, use a custom middleware or view mixin to handle the override pattern.

---

#### Task Controller (`src/controllers/TaskController.js` or `tasks/views.py`)

**Responsibility:** Orchestrate the request-response cycle for each task operation. Calls the validation layer, the ORM, and the template engine. Handles error conditions and sets appropriate HTTP status codes.

**Actions and their contracts:**

- **`list(req, res)`** — Read `?status=` query param → map slug to enum → query `Task.findAll({where, order})` → render `tasks/index` template → HTTP 200; on DB error → HTTP 500
- **`new(req, res)`** — Render `tasks/new` template with empty form → HTTP 200
- **`create(req, res)`** — Extract + trim form fields → `validateTaskInput()` → on failure: re-render `tasks/new` with errors HTTP 422 → on success: `Task.create({...})` → redirect `/tasks` HTTP 302; on DB error → HTTP 500
- **`show(req, res)`** — Parse `:id` integer → `Task.findByPk(id)` → on not found: HTTP 404 → render `tasks/show` → HTTP 200
- **`edit(req, res)`** — Parse `:id` integer → `Task.findByPk(id)` → on not found: HTTP 404 → render `tasks/edit` pre-populated → HTTP 200
- **`update(req, res)`** — Parse `:id` → find task → `validateTaskInput()` → on failure: re-render `tasks/edit` with errors HTTP 422 → on success: `task.update({...})` → redirect `/tasks` HTTP 302
- **`destroy(req, res)`** — Parse `:id` → find task → `task.destroy()` → redirect `/tasks` HTTP 302; on not found: HTTP 404

---

#### Validation Layer (`src/validation/taskValidation.js` or `tasks/validators.py`)

**Responsibility:** Pure functions that validate task input fields and return structured error objects. Stateless — no database access.

**Function signature:**

```typescript
interface ValidationError {
  field: string;       // e.g., "title", "due_date", "status"
  code: string;        // e.g., "TITLE_REQUIRED"
  message: string;     // e.g., "Title is required."
}

function validateTaskInput(
  input: TaskFormInput,
  context: 'create' | 'edit'
): ValidationError[];
```

**Rules applied:**
- `title`: trim → check non-empty → check ≤ 255 chars
- `status` (edit only): must be one of `['To Do', 'In Progress', 'Done']`
- `due_date`: if non-empty, parse as `YYYY-MM-DD` → validate it's a real calendar date
- `description`: no validation (accept as-is, empty → NULL)

Returns empty array `[]` if all fields are valid.

---

#### Task Model (`src/models/Task.js` or `tasks/models.py`)

**Responsibility:** ORM model definition for the `tasks` table. Defines column types, constraints, default values, and hooks. Exposes Sequelize/Django ORM query methods to controllers.

**Key model properties:**
- Maps directly to the `tasks` table (see §3 Data Model)
- `status` field uses a string enum validated at ORM level in addition to DB CHECK constraint
- `updatedAt` / `updated_at` is automatically managed by Sequelize/Django ORM on every `save()`/`update()`

---

#### Template Engine (`src/views/` or `templates/`)

**Responsibility:** Render server-side HTML for all pages. Must auto-escape all interpolated values to prevent XSS. No client-side rendering framework.

**Templates required:**

| Template | Route | Purpose |
|----------|-------|---------|
| `tasks/index` | `GET /tasks` | Task list with filter controls |
| `tasks/new` | `GET /tasks/new` | Task creation form |
| `tasks/show` | `GET /tasks/:id` | Read-only task detail |
| `tasks/edit` | `GET /tasks/:id/edit` | Task edit form (pre-populated) |
| `errors/400` | — | Bad request error page |
| `errors/404` | — | Not found error page |
| `errors/500` | — | Internal server error page |
| `errors/503` | — | Service unavailable (DB down) |
| `layouts/base` | — | Shared HTML shell (head, nav, footer) |

---

### 2.2 Frontend Components

TaskFlow v1 uses **server-rendered HTML only**. There is no JavaScript framework, no build step, and no SPA routing. JavaScript is used minimally, only for the deletion confirmation dialog.

#### Task List Page (`tasks/index`)

- Renders filter control tabs/buttons: `All`, `To Do`, `In Progress`, `Done`
- Active filter tab is visually distinguished (bold/filled style)
- Each filter option is a plain `<a href="/tasks?status={slug}">` link — no JS required
- Task rows display: title (link to detail), status badge, due date (formatted `MMM DD, YYYY`), description preview (≤100 chars + `…`)
- "New Task" button links to `GET /tasks/new`
- Delete button on each row triggers confirmation before submitting a form to `POST /tasks/:id/delete`
- Empty state message shown when no tasks match

#### Task Forms (`tasks/new`, `tasks/edit`)

- Standard HTML `<form>` with `method="POST"`
- Edit form uses a hidden `<input name="_method" value="PUT">` for method override
- Status field is a `<select>` with three `<option>` elements
- Due date field is `<input type="date">`
- Validation errors rendered as inline `<span class="error">` adjacent to each invalid field
- Form fields re-populated with submitted values on validation failure

#### Confirmation Dialog (Delete)

- Options:
  - **Option A (recommended):** Inline `<form method="POST" action="/tasks/:id/delete">` with a `window.confirm()` call on the submit button's `onclick` handler
  - **Option B:** A lightweight vanilla JS modal (no library dependency)
  - Either approach is compliant; implementation choice is left to the developer

---

## 3. Data Model

### 3.1 Entity Overview

TaskFlow v1 has a **single domain entity**: `Task`. There are no user tables, session tables, or relationship tables. All tasks exist in a single shared workspace.

```
┌─────────────────────────────────────────────┐
│                    tasks                    │
├─────────────────────────────────────────────┤
│  id          SERIAL          PK             │
│  title       VARCHAR(255)    NOT NULL       │
│  description TEXT            NULL           │
│  status      VARCHAR(20)     NOT NULL       │
│              DEFAULT 'To Do'                │
│              CHECK IN ('To Do',             │
│                        'In Progress',       │
│                        'Done')              │
│  due_date    DATE            NULL           │
│  created_at  TIMESTAMPTZ     NOT NULL       │
│              DEFAULT NOW()                  │
│  updated_at  TIMESTAMPTZ     NOT NULL       │
│              DEFAULT NOW()                  │
└─────────────────────────────────────────────┘
```

There are no foreign keys in v1. Future v2 candidates (not implemented): `assigned_to` (FK → users), `project_id` (FK → projects), `deleted_at` (soft-delete).

---

### 3.2 Complete DDL (PostgreSQL)

```sql
-- ============================================================
-- TaskFlow v1 — PostgreSQL DDL
-- Migration: 001_create_tasks_table
-- Run via: npx sequelize-cli db:migrate
--          OR python manage.py migrate
-- ============================================================

-- Drop existing table for clean migration (development only)
-- DROP TABLE IF EXISTS tasks;

CREATE TABLE tasks (
    id          SERIAL                      PRIMARY KEY,
    title       VARCHAR(255)                NOT NULL,
    description TEXT                        NULL,
    status      VARCHAR(20)                 NOT NULL
                    DEFAULT 'To Do'
                    CONSTRAINT tasks_status_check
                        CHECK (status IN ('To Do', 'In Progress', 'Done')),
    due_date    DATE                        NULL,
    created_at  TIMESTAMP WITH TIME ZONE    NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP WITH TIME ZONE    NOT NULL DEFAULT NOW()
);

-- ============================================================
-- Indexes
-- ============================================================

-- Supports: WHERE status = $1 (F01 status filter, F05)
CREATE INDEX idx_tasks_status
    ON tasks (status);

-- Supports: ORDER BY created_at DESC (default sort on task list)
CREATE INDEX idx_tasks_created_at
    ON tasks (created_at DESC);

-- ============================================================
-- Comments
-- ============================================================

COMMENT ON TABLE tasks IS
    'Core domain table. Stores all tasks in the shared team workspace. Single-table design for v1.';

COMMENT ON COLUMN tasks.id IS
    'System-assigned auto-incrementing primary key. Never set by application code. Never reused after deletion.';

COMMENT ON COLUMN tasks.title IS
    'Human-readable task name. Required. Max 255 chars. Validated at application layer (non-empty after trim) and enforced at DB layer (NOT NULL).';

COMMENT ON COLUMN tasks.description IS
    'Optional free-form multi-line text. NULL when not provided or cleared. No length limit at DB layer.';

COMMENT ON COLUMN tasks.status IS
    'Task state enum. One of: ''To Do'' (default), ''In Progress'', ''Done''. Enforced at DB level via CHECK constraint and at application level via validation.';

COMMENT ON COLUMN tasks.due_date IS
    'Optional intended completion date. DATE type (no time component). NULL when not provided. Stored and queried as calendar date only.';

COMMENT ON COLUMN tasks.created_at IS
    'UTC timestamp of row insertion. Set once at INSERT by DB DEFAULT. Never modified by UPDATE operations.';

COMMENT ON COLUMN tasks.updated_at IS
    'UTC timestamp of last modification. Set at INSERT by DB DEFAULT. Must be updated to NOW() on every UPDATE operation (handled by ORM).';
```

---

### 3.3 SQLite DDL (Development/Testing)

The ORM generates this automatically from the model definition when `DATABASE_URL` points to a SQLite file. Shown here for reference:

```sql
-- SQLite equivalent (auto-generated by Sequelize/Django ORM)
CREATE TABLE IF NOT EXISTS "tasks" (
    "id"          INTEGER         PRIMARY KEY AUTOINCREMENT,
    "title"       VARCHAR(255)    NOT NULL,
    "description" TEXT            NULL,
    "status"      VARCHAR(20)     NOT NULL DEFAULT 'To Do'
                  CHECK ("status" IN ('To Do', 'In Progress', 'Done')),
    "due_date"    DATE            NULL,
    "created_at"  DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updated_at"  DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS "idx_tasks_status"     ON "tasks" ("status");
CREATE INDEX IF NOT EXISTS "idx_tasks_created_at" ON "tasks" ("created_at" DESC);
```

---

### 3.4 Column Definitions Reference

| Column | PostgreSQL Type | SQLite Type | Nullable | Default | Constraints | Notes |
|--------|----------------|-------------|----------|---------|-------------|-------|
| `id` | `SERIAL` | `INTEGER AUTOINCREMENT` | NOT NULL | auto | Primary Key | Never set by app code; never reused |
| `title` | `VARCHAR(255)` | `VARCHAR(255)` | NOT NULL | — | ≤ 255 chars | App validates non-empty after trim |
| `description` | `TEXT` | `TEXT` | NULL | NULL | — | NULL when not provided or cleared |
| `status` | `VARCHAR(20)` | `VARCHAR(20)` | NOT NULL | `'To Do'` | CHECK enum | `'To Do'` \| `'In Progress'` \| `'Done'` |
| `due_date` | `DATE` | `DATE` | NULL | NULL | Valid calendar date | No time component; `YYYY-MM-DD` format |
| `created_at` | `TIMESTAMP WITH TIME ZONE` | `DATETIME` | NOT NULL | `NOW()` | — | Set at INSERT; never modified |
| `updated_at` | `TIMESTAMP WITH TIME ZONE` | `DATETIME` | NOT NULL | `NOW()` | — | Updated on every UPDATE by ORM |

---

### 3.5 Query Patterns

All queries are executed via ORM; raw SQL is never used directly. These patterns document the logical intent:

```sql
-- F01: List all tasks (unfiltered)
SELECT id, title, description, status, due_date, created_at
FROM tasks
ORDER BY created_at DESC;

-- F01/F05: List tasks filtered by status
SELECT id, title, description, status, due_date, created_at
FROM tasks
WHERE status = $1          -- parameterized: 'To Do' | 'In Progress' | 'Done'
ORDER BY created_at DESC;

-- F00: Create task
INSERT INTO tasks (title, description, status, due_date, created_at, updated_at)
VALUES ($1, $2, $3, $4, NOW(), NOW())
RETURNING id;

-- F02: Fetch single task for edit/show
SELECT * FROM tasks WHERE id = $1;

-- F02: Update task
UPDATE tasks
SET title = $1,
    description = $2,
    status = $3,
    due_date = $4,
    updated_at = NOW()
WHERE id = $5;

-- F03: Delete task (hard delete)
DELETE FROM tasks WHERE id = $1;
```

---

## 4. API Design

### 4.1 TypeScript Interfaces

These interfaces define the canonical data shapes for the `Task` entity across all API operations. The application server may be Node.js (TypeScript) or Django (Python); Python equivalents use dataclasses or Pydantic models.

```typescript
// ============================================================
// Core Domain Type
// ============================================================

type TaskStatus = 'To Do' | 'In Progress' | 'Done';

type StatusSlug = 'todo' | 'in_progress' | 'done' | 'all';

interface Task {
  id: number;
  title: string;
  description: string | null;
  status: TaskStatus;
  due_date: string | null;    // 'YYYY-MM-DD' or null
  created_at: string;         // ISO 8601 UTC: 'YYYY-MM-DDTHH:MM:SSZ'
  updated_at: string;         // ISO 8601 UTC: 'YYYY-MM-DDTHH:MM:SSZ'
}

// ============================================================
// Form Input Types (from HTML form submissions)
// ============================================================

interface TaskCreateInput {
  title: string;              // required; validated non-empty after trim
  description?: string;       // optional; empty string → null
  status?: string;            // optional; invalid/missing → defaults to 'To Do'
  due_date?: string;          // optional; 'YYYY-MM-DD' or empty string → null
}

interface TaskUpdateInput {
  title: string;              // required
  description?: string;       // optional; empty string → null (clears field)
  status: string;             // required on edit
  due_date?: string;          // optional; empty string → null (clears field)
  _method?: 'PUT';            // HTML method override (ignored by server logic)
}

// ============================================================
// Validation Error Shape
// ============================================================

interface ValidationError {
  field: string;              // 'title' | 'due_date' | 'status'
  code: string;               // error code constant (see Y2 error catalog)
  message: string;            // user-facing message
}

interface ValidationResult {
  valid: boolean;
  errors: ValidationError[];
}

// ============================================================
// Controller Response Helpers
// ============================================================

interface TaskListViewModel {
  tasks: Task[];
  activeFilter: StatusSlug | null;  // null means 'all'
  filterOptions: Array<{
    slug: StatusSlug | null;
    label: string;
    href: string;
    active: boolean;
  }>;
  emptyStateMessage: string | null; // null when tasks.length > 0
}

interface TaskFormViewModel {
  task: Partial<Task>;          // pre-populated values (empty on create)
  errors: ValidationError[];    // empty array on initial render
  formAction: string;           // '/tasks' (create) or '/tasks/:id' (edit)
  formMethod: 'POST';           // always POST (method override for PUT)
  hiddenMethod?: 'PUT';         // present on edit form only
}
```

---

### 4.2 Endpoint Catalog

#### `GET /`

| Property | Value |
|----------|-------|
| **Purpose** | Root URL entry point |
| **Response** | `302 Redirect → /tasks` |
| **Auth** | None |

---

#### `GET /tasks`

| Property | Value |
|----------|-------|
| **Purpose** | Render task list page; optionally filtered by status |
| **Auth** | None |

**Query Parameters:**

| Parameter | Type | Required | Values | Default |
|-----------|------|----------|--------|---------|
| `status` | string | No | `todo`, `in_progress`, `done`, `all` | Show all (no filter) |

**Responses:**

| HTTP Status | Condition | Body |
|-------------|-----------|------|
| `200 OK` | Success — tasks found or list is empty | HTML task list page |
| `500 Internal Server Error` | Database read failure | HTML error page |

**Slug → Enum Mapping:**

| `?status=` value | DB `WHERE status =` | Active filter label |
|------------------|---------------------|---------------------|
| (absent) | (no WHERE clause) | All |
| `all` | (no WHERE clause) | All |
| `todo` | `'To Do'` | To Do |
| `in_progress` | `'In Progress'` | In Progress |
| `done` | `'Done'` | Done |
| (unknown value) | (no WHERE clause) | All |

---

#### `GET /tasks/new`

| Property | Value |
|----------|-------|
| **Purpose** | Render blank task creation form |
| **Auth** | None |

**Responses:**

| HTTP Status | Condition | Body |
|-------------|-----------|------|
| `200 OK` | Always | HTML creation form; status defaulted to `To Do` |

---

#### `POST /tasks`

| Property | Value |
|----------|-------|
| **Purpose** | Create a new task |
| **Content-Type** | `application/x-www-form-urlencoded` |
| **Auth** | None |

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | **Yes** | Non-empty after trim; ≤ 255 chars |
| `description` | string | No | Any text; empty → NULL |
| `status` | string | No | `'To Do'` \| `'In Progress'` \| `'Done'`; invalid/absent → `'To Do'` |
| `due_date` | string | No | `YYYY-MM-DD` format or empty; empty → NULL |

**Responses:**

| HTTP Status | Condition | Body |
|-------------|-----------|------|
| `302 Found` | Task created | Redirect → `GET /tasks` |
| `422 Unprocessable Entity` | Validation failure | HTML creation form with inline errors; submitted values pre-populated |
| `500 Internal Server Error` | Database write failure | HTML error page |

**Error Codes:**

| Code | Field | Trigger |
|------|-------|---------|
| `TITLE_REQUIRED` | `title` | Empty or whitespace-only after trim |
| `TITLE_TOO_LONG` | `title` | Exceeds 255 characters |
| `DUE_DATE_INVALID` | `due_date` | Non-empty but not a valid `YYYY-MM-DD` date |

---

#### `GET /tasks/:id`

| Property | Value |
|----------|-------|
| **Purpose** | Render read-only task detail view |
| **Auth** | None |

**Path Parameters:**

| Parameter | Type | Required | Constraint |
|-----------|------|----------|------------|
| `id` | integer | **Yes** | Positive integer |

**Responses:**

| HTTP Status | Condition | Body |
|-------------|-----------|------|
| `200 OK` | Task found | HTML task detail page |
| `400 Bad Request` | `:id` is not a positive integer | HTML 400 error page |
| `404 Not Found` | Task does not exist | HTML 404 error page |

---

#### `GET /tasks/:id/edit`

| Property | Value |
|----------|-------|
| **Purpose** | Render task edit form pre-populated with current values |
| **Auth** | None |

**Path Parameters:**

| Parameter | Type | Required | Constraint |
|-----------|------|----------|------------|
| `id` | integer | **Yes** | Positive integer |

**Responses:**

| HTTP Status | Condition | Body |
|-------------|-----------|------|
| `200 OK` | Task found | HTML edit form with fields pre-populated |
| `400 Bad Request` | `:id` is not a positive integer | HTML 400 error page |
| `404 Not Found` | Task does not exist | HTML 404 error page |

---

#### `PUT /tasks/:id` — also `POST /tasks/:id` with `_method=PUT`

| Property | Value |
|----------|-------|
| **Purpose** | Update an existing task |
| **Content-Type** | `application/x-www-form-urlencoded` |
| **Auth** | None |

**Path Parameters:**

| Parameter | Type | Required | Constraint |
|-----------|------|----------|------------|
| `id` | integer | **Yes** | Positive integer referencing existing row |

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | **Yes** | Non-empty after trim; ≤ 255 chars |
| `description` | string | No | Any text; empty string → NULL (clears field) |
| `status` | string | **Yes** | Must be `'To Do'` \| `'In Progress'` \| `'Done'` |
| `due_date` | string | No | `YYYY-MM-DD` or empty; empty → NULL (clears field) |
| `_method` | string | No | `'PUT'` — method override for HTML form |

**Responses:**

| HTTP Status | Condition | Body |
|-------------|-----------|------|
| `302 Found` | Task updated | Redirect → `GET /tasks` |
| `400 Bad Request` | `:id` not a positive integer | HTML 400 error page |
| `404 Not Found` | Task does not exist | HTML 404 error page |
| `422 Unprocessable Entity` | Validation failure | HTML edit form with inline errors; submitted values pre-populated |
| `500 Internal Server Error` | Database write failure | HTML error page |

**Error Codes:**

| Code | Field | Trigger |
|------|-------|---------|
| `TITLE_REQUIRED` | `title` | Empty or whitespace-only after trim |
| `TITLE_TOO_LONG` | `title` | Exceeds 255 characters |
| `DUE_DATE_INVALID` | `due_date` | Non-empty but not a valid `YYYY-MM-DD` date |
| `STATUS_INVALID` | `status` | Missing or not one of the three valid values |
| `INVALID_TASK_ID` | `:id` | Not a positive integer |

---

#### `DELETE /tasks/:id` — also `POST /tasks/:id/delete`

| Property | Value |
|----------|-------|
| **Purpose** | Permanently delete a task (hard delete) |
| **Auth** | None |

**Path Parameters:**

| Parameter | Type | Required | Constraint |
|-----------|------|----------|------------|
| `id` | integer | **Yes** | Positive integer referencing existing row |

**Request Body (for `POST /tasks/:id/delete` form override only):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `_method` | string | No | `'DELETE'` — method override for HTML form |

**Responses:**

| HTTP Status | Condition | Body |
|-------------|-----------|------|
| `302 Found` | Task deleted | Redirect → `GET /tasks` |
| `400 Bad Request` | `:id` not a positive integer | HTML 400 error page |
| `404 Not Found` | Task does not exist (already deleted or never existed) | HTML 404 error page |
| `500 Internal Server Error` | Database delete failure | HTML error page |

**Error Codes:**

| Code | Trigger |
|------|---------|
| `TASK_NOT_FOUND` | `:id` references no existing row |
| `INVALID_TASK_ID` | `:id` is not a positive integer |
| `DB_DELETE_ERROR` | Database DELETE statement fails |

---

### 4.3 Full Error Code Reference

| Code | HTTP Status | Feature(s) | Trigger | User-Facing Message |
|------|-------------|------------|---------|---------------------|
| `TITLE_REQUIRED` | 422 | F00, F02 | `title` empty/whitespace after trim | "Title is required." |
| `TITLE_TOO_LONG` | 422 | F00, F02 | `title` > 255 chars | "Title must be 255 characters or fewer." |
| `DUE_DATE_INVALID` | 422 | F00, F02 | `due_date` non-empty and not valid `YYYY-MM-DD` | "Due date must be a valid date (YYYY-MM-DD)." |
| `STATUS_INVALID` | 422 | F02 | `status` missing or not in enum (edit only) | "Status must be one of: To Do, In Progress, Done." |
| `INVALID_TASK_ID` | 400 | F02, F03 | `:id` not a positive integer | "Invalid task ID." |
| `TASK_NOT_FOUND` | 404 | F02, F03 | `:id` has no matching `tasks` row | "Task not found." / "Task not found. It may have already been deleted." |
| `DB_READ_ERROR` | 500 | F01, F05 | DB query fails on list/detail read | "Unable to load tasks. Please try again." |
| `DB_WRITE_ERROR` | 500 | F00, F02 | DB INSERT/UPDATE fails after validation | "Unable to save the task. Please try again." |
| `DB_DELETE_ERROR` | 500 | F03 | DB DELETE fails | "Unable to delete the task. Please try again." |
| `DB_CONNECTION_ERROR` | 503 | F04 (all) | DB unreachable at request time | "The application is temporarily unavailable. Please try again later." |
| `DB_SCHEMA_MISSING` | 500 | F04 | `tasks` table doesn't exist (migration not run) | "Database schema not initialized. Contact the administrator." |

---

## 5. Security Architecture

### 5.1 Authentication & Authorization

**Authentication:** None in v1. TaskFlow is a single shared workspace with zero per-user identity. There is no login, no session, no token, and no cookie (beyond any default Express session cookie, which should be disabled). Every HTTP request from any browser has full read/write/delete access.

**Authorization:** Not applicable in v1. All users (any browser with network access to the server) can perform all CRUD operations on all tasks.

**Accepted risk:** Any team member can delete all tasks. This is mitigated by the mandatory confirmation dialog on delete (F03). Soft-delete and undo are v1.1 candidates.

**Access control boundary:** The application should be deployed behind a firewall or VPN such that only internal team members can reach the service. This is an operations/network concern, not an application concern.

---

### 5.2 Input Validation & Sanitization

**Application-level validation (before any DB operation):**

- All text inputs are trimmed before validation
- `title` is validated as non-empty and ≤ 255 characters
- `due_date` is validated as a real calendar date if non-empty
- `status` is validated as one of three known enum values on edit
- `:id` path parameters are validated as positive integers before any DB query

**Database-level constraints (defense in depth):**

- `title NOT NULL` prevents null row insertion even if app validation fails
- `status CHECK (status IN (...))` rejects invalid enum values at the DB layer
- `SERIAL` primary key — the DB controls ID assignment; application code never writes `id`

**ORM-level protection:**

- All queries use ORM parameterized methods (`.create()`, `.update()`, `.findByPk()`) — never raw string-concatenated SQL
- SQL injection is structurally impossible through correct ORM usage
- Sequelize and Django ORM both use prepared statements under the hood

---

### 5.3 Output Encoding & XSS Prevention

**Requirement:** All user-supplied content (task `title`, `description`, `status`) must be HTML-escaped before being rendered into any template. A single unescaped `title` containing `<script>alert(1)</script>` must never execute in any browser.

**Implementation:**

- **EJS (Express):** Use `<%= value %>` (escaped) — never `<%- value %>` (raw) for user content
- **Jinja2 (Django):** Auto-escaping is enabled by default — never use `{{ value | safe }}` for user content
- **Handlebars (Express):** Use `{{ value }}` (escaped) — never use `{{{ value }}}` for user content

**Rule:** No template should render user-controlled content as raw/unescaped HTML. Template authors must default to the escaped interpolation syntax.

---

### 5.4 HTTP Security Headers

The following HTTP response headers should be set on all responses (can be added via Express `helmet` middleware or Django's `SecurityMiddleware`):

| Header | Value | Purpose |
|--------|-------|---------|
| `X-Content-Type-Options` | `nosniff` | Prevent MIME-type sniffing |
| `X-Frame-Options` | `SAMEORIGIN` | Prevent clickjacking |
| `X-XSS-Protection` | `1; mode=block` | Legacy XSS filter (belt-and-suspenders) |
| `Referrer-Policy` | `same-origin` | Limit referrer leakage |

**Note:** CSP (`Content-Security-Policy`) is recommended but omitted from the mandatory set for v1 to avoid blocking inline scripts used for the delete confirmation dialog.

---

### 5.5 Error Information Leakage

**Rule:** Error pages and API responses must never expose:
- Database connection strings or credentials
- File system paths
- Stack traces (visible in browser)
- Internal database error messages (e.g., constraint violation text)
- Environment variable names or values

**Implementation:**
- Log full error details (stack trace, DB error) server-side only
- Render only the user-facing message string from the error catalog (§4.3) to the browser
- Use a generic error page template that accepts a `message` string parameter

---

### 5.6 Database Connection Security

| Concern | Mitigation |
|---------|------------|
| Credentials in source code | `DATABASE_URL` loaded from environment variable; never hardcoded or committed |
| Plain-text PostgreSQL connections | SSL recommended (`?sslmode=require`) for production; not enforced for SQLite |
| Connection pool exhaustion | Pool capped at 5–10 connections; adequate for ≤15 concurrent users |
| DB port exposed to internet | PostgreSQL should bind to `localhost` or private network only |

---

## 6. Technology Stack

### 6.1 Stack Table

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| **Runtime** | Node.js | 18 LTS+ | JavaScript execution environment |
| **Web Framework** | Express.js | 4.x | HTTP routing, middleware, request/response |
| **ORM** | Sequelize | 6.x | Parameterized queries, connection pooling, migrations |
| **Migration CLI** | sequelize-cli | 6.x | `db:migrate` command for schema setup |
| **Template Engine** | EJS or Handlebars | Latest | Server-rendered HTML with auto-escaping |
| **Method Override** | method-override | 3.x | HTML form `_method` → PUT/DELETE |
| **Security Headers** | helmet | 7.x | Sets HTTP security headers automatically |
| **Database (prod)** | PostgreSQL | 14+ | Relational persistence, ACID writes |
| **Database (dev)** | SQLite | 3.x | Local development, zero setup |
| **DB Driver (PG)** | pg | 8.x | Node.js PostgreSQL client |
| **DB Driver (SQLite)** | sqlite3 | 5.x | Node.js SQLite client |
| **Package Manager** | npm | 9+ | Dependency management |

**Alternative Python/Django stack** (either stack is compliant — implementation choice):

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| **Runtime** | Python | 3.10+ | Execution environment |
| **Web Framework** | Django | 4.x | Full-stack web framework |
| **ORM** | Django ORM | (built-in) | Query abstraction, migrations |
| **Migration CLI** | Django `manage.py migrate` | (built-in) | Schema setup |
| **Template Engine** | Django Templates / Jinja2 | (built-in) | Server-rendered HTML (auto-escaping on) |
| **Database (prod)** | PostgreSQL | 14+ | Relational persistence |
| **Database (dev)** | SQLite | 3.x | Local development |
| **Security** | Django `SecurityMiddleware` | (built-in) | HTTPS redirect, security headers |
| **Package Manager** | pip + requirements.txt | — | Dependency management |

---

### 6.2 Dependency Philosophy

- **Zero exotic dependencies** — every package chosen is the de-facto standard for its purpose
- **No client-side JS frameworks** — no React, Vue, Angular, Svelte, or Alpine required
- **No build pipeline** — no webpack, Vite, Babel, or TypeScript compilation step at runtime
- **No caching layer** — no Redis, Memcached; direct DB reads on every request
- **No message queue** — no RabbitMQ, SQS; all operations are synchronous request/response
- **No authentication library** — no Passport.js, Auth0, JWT; no auth in v1

---

### 6.3 Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | **Yes** | — | DB connection string. Format: `postgres://user:pass@host:port/dbname` or `sqlite:///./taskflow.db` |
| `PORT` | No | `3000` (Node) / `8000` (Django) | HTTP port the server listens on |
| `NODE_ENV` | No | `development` | `production` disables verbose error output |

---

## 7. Integration Points

### 7.1 Integration Overview

TaskFlow v1 has **no external service calls**. All integrations are local:

| # | System | Type | Required | Direction |
|---|--------|------|----------|-----------|
| 1 | PostgreSQL / SQLite | Data store | Yes | App → DB |
| 2 | Node.js / Python Runtime | Execution environment | Yes | — (process) |
| 3 | Sequelize / Django ORM | Library (bundled) | Yes | App → ORM → DB |
| 4 | Web Browser | Client | Yes | Browser ↔ App |

There are no integrations with: email services, push notifications, OAuth providers, external authentication, object storage, CDNs, analytics services, monitoring platforms, or WebSocket servers.

---

### 7.2 Database Integration

**System:** PostgreSQL 14+ (production), SQLite 3.x (development)  
**Protocol:** TCP socket (PostgreSQL), local file I/O (SQLite)  
**Configuration:** `DATABASE_URL` environment variable  

**Connection behavior:**
- Application establishes connection pool on startup (5–10 connections for PostgreSQL)
- Application performs a health check query (`SELECT 1`) on startup; fails fast with a clear error if DB is unreachable
- All queries are parameterized through the ORM — no string-concatenated SQL
- If the DB becomes unreachable at request time, the application returns HTTP 503 and logs the full error

**Environment switching:** Switching from SQLite (dev) to PostgreSQL (prod) requires only changing `DATABASE_URL`. No code changes are needed. The ORM handles dialect differences automatically.

**Migration flow:**
```bash
# Initial setup (creates tasks table + indexes)
npx sequelize-cli db:migrate        # Node.js
python manage.py migrate            # Django

# Verify schema was created
npx sequelize-cli db:migrate:status
python manage.py showmigrations
```

---

### 7.3 Browser Integration

**System:** Chrome, Firefox, Safari, Edge (current versions)  
**Protocol:** HTTP/1.1  
**Content-Type:** `text/html; charset=utf-8`  

**Client-side behavior:**
- All rendering is server-side; browser receives complete HTML documents
- Forms use standard `method="POST"` with `application/x-www-form-urlencoded` encoding
- PUT and DELETE are achieved via a hidden `_method` field in POST body (interpreted by `method-override` middleware)
- JavaScript is used minimally — only for the delete confirmation dialog (`window.confirm()` or a lightweight vanilla modal)
- No cookies, no session storage, no localStorage used by the application
- No client-side routing; every navigation is a full page load

**Browser compatibility requirements:** All features must work in current versions of Chrome, Firefox, Safari, and Edge without any polyfills or transpilation.

---

### 7.4 No Third-Party Integrations (v1)

The following are explicitly out of scope and must not be added to v1:

| Service Type | Examples | Status |
|-------------|----------|--------|
| Email / Notifications | SendGrid, Mailgun, Slack webhooks | ❌ Out of scope |
| Authentication | Auth0, Okta, Passport.js, Google OAuth | ❌ Out of scope |
| Object Storage | S3, GCS, Cloudinary | ❌ Out of scope |
| Analytics | Segment, Mixpanel, Google Analytics | ❌ Out of scope |
| Error Tracking | Sentry, Rollbar, Datadog | ❌ Out of scope (server logs only) |
| CDN | Cloudflare, Fastly | ❌ Out of scope |
| WebSockets | Socket.io, Pusher | ❌ Out of scope |

---

## Appendix A: Architectural Decision Log

| Decision | Options Considered | Choice | Rationale |
|----------|--------------------|--------|-----------|
| Architecture pattern | Monolith, Microservices, SPA + API | Monolith (server-rendered) | Single developer/small team; no build pipeline; readable in < 1 hour |
| No authentication | Full auth, magic links, shared password | No auth | Single shared workspace; login adds friction without value for ≤15 people; v2 candidate |
| PostgreSQL + SQLite | PostgreSQL only, MongoDB, MySQL | Postgres (prod) + SQLite (dev) | Relational fit for CRUD; SQLite zero-config for local dev; single env var to switch |
| Server-rendered HTML | React SPA, Vue SPA, HTMX | Plain server-rendered | No build step; no JS framework; works without JS (accessibility); simpler mental model |
| Hard delete | Soft delete, archive | Hard delete | Simpler schema; simpler queries; confirmation dialog mitigates accident risk; soft-delete is v1.1 |
| Status as VARCHAR + CHECK | ENUM type, separate table | VARCHAR + CHECK | PostgreSQL ENUM requires DDL change to add values; VARCHAR + CHECK is portable to SQLite |
| ORM (no raw SQL) | Raw SQL, query builder, ORM | ORM | Parameterized queries by default (SQL injection prevention); migration management; dialect portability |

---

## Appendix B: File Structure Reference

```
taskflow/
├── src/                          # Node.js / Express source
│   ├── app.js                    # Express application setup
│   ├── server.js                 # HTTP server entry point
│   ├── routes/
│   │   └── tasks.js              # Route definitions
│   ├── controllers/
│   │   └── TaskController.js     # Request handlers
│   ├── models/
│   │   └── Task.js               # Sequelize model definition
│   ├── validation/
│   │   └── taskValidation.js     # Pure validation functions
│   └── views/                    # EJS templates
│       ├── layouts/
│       │   └── base.ejs          # Shared HTML shell
│       ├── tasks/
│       │   ├── index.ejs         # Task list page
│       │   ├── new.ejs           # Creation form
│       │   ├── show.ejs          # Task detail
│       │   └── edit.ejs          # Edit form
│       └── errors/
│           ├── 400.ejs
│           ├── 404.ejs
│           ├── 500.ejs
│           └── 503.ejs
├── migrations/
│   └── 001-create-tasks.js       # Initial schema migration
├── config/
│   └── database.js               # Sequelize DB config (reads DATABASE_URL)
├── package.json
├── .env.example                  # Example environment variables
└── README.md
```

---

*Document generated: 2026-07-01 | TechArch-TaskFlow v1.0*  
*Based on: PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, PROJECT.md v1.0*
