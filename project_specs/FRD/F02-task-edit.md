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
