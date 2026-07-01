---

## Flow 01: View & Filter Task List

**Trigger:** User navigates to `GET /tasks` (direct URL, bookmark, redirect after CRUD operation)
**User Stories:** US-1.1, US-1.2, US-1.3, US-5.1, US-5.2, US-5.3, US-5.4
**Personas:** PER-02 Priya Nair (primary — JRN-02.1, JRN-02.2), PER-01 Marcus Webb (secondary — JRN-01.2)

```
[Browser — navigate to /tasks or /tasks?status={slug}]
        │
        ▼
  ┌─────────────────────────────┐
  │  Server reads ?status= param │
  │  Queries tasks table         │
  └─────────────────────────────┘
        │
        ├── DB READ ERROR ──▶ HTTP 500 ──▶ [Error Page]
        │                                  "Unable to load tasks.
        │                                   Please try again."
        │
        └── SUCCESS ──▶ HTTP 200 ──▶ [Task List Page]
                                       │
                                       ├── Tasks found ──▶ render task rows
                                       │
                                       └── No tasks ──▶ render empty state
                                                         message + "New Task" btn

                                   [Task List Page — user interactions]
                                       │
                                       ├── click filter tab ──▶ GET /tasks?status={slug}
                                       │                         (same flow, URL updated)
                                       │
                                       ├── click task title ──▶ GET /tasks/:id/edit
                                       │                         (→ Flow-02: Edit Task)
                                       │
                                       ├── click "New Task" ──▶ GET /tasks/new
                                       │                         (→ Flow-00: Create Task)
                                       │
                                       └── click "Delete" ──▶ confirmation dialog
                                                               (→ Flow-03: Delete Task)
```

### Steps

1. **Arrive:** User loads `/tasks` (unfiltered) or `/tasks?status={slug}` (filtered). Priya typically arrives via bookmark to her pre-filtered "In Progress" view (JRN-02.1 Arrive stage).
2. **Filter Controls Render:** Four filter tabs/buttons always visible above the list: **All** | **To Do** | **In Progress** | **Done**. The active filter is visually distinguished (filled/bold style). "All" is active when no `?status=` param is present.
3. **Task Rows Render:** Each row shows: task title (clickable link), status badge, due date (`MMM DD, YYYY` format or blank), description preview (first 100 chars + `…`).
4. **Apply Filter:** User clicks a filter tab → browser navigates to the corresponding URL. The filter state is URL-driven (bookmarkable, shareable, refresh-safe — US-5.3).
5. **Navigate to Task:** User clicks a task title → navigates to `GET /tasks/:id/edit` to view full details and edit.
6. **Empty State:** If no tasks match (filtered or unfiltered), an appropriate empty state message is shown. Filter controls and "New Task" button remain visible.

### Key UX Decisions

- **Filter tabs above the list**, not in a sidebar — single row of 4 options, easy to discover (1 click to filter — JRN-01.2 Filter stage requirement).
- **Due date visible on every row** — Priya needs to spot overdue tasks inline without clicking (JRN-02.1 Spot Risk, JRN-02.2 Scan Due Dates).
- **Task title is the primary clickable element** — clicking the title is the primary navigation to edit/detail view (US-1.2).
- **Edit and Delete actions** are secondary, accessible per-row (e.g., small action icons or links at the row end).
- **Newest-first sort** is always applied — no sort toggle needed for v1.
