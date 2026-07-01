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
