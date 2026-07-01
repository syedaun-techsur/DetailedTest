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
