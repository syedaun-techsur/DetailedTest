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
