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
