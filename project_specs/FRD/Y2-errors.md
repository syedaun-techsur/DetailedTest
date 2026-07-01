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
