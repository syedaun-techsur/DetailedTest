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
