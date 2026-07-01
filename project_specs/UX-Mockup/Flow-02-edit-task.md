---

## Flow 02: Edit Task

**Trigger:** User clicks task title or "Edit" action on the Task List
**User Stories:** US-2.1, US-2.2, US-2.3, US-2.4
**Personas:** PER-01 Marcus Webb (status updates — JRN-01.3), PER-02 Priya Nair (curation — JRN-02.3)

```
[Task List — GET /tasks]
        │
        │  click task title or "Edit" link on row
        ▼
  ┌─────────────────────────────────────┐
  │  GET /tasks/:id/edit                 │
  │  Server looks up task by :id         │
  └─────────────────────────────────────┘
        │
        ├── :id invalid (not int) ──▶ HTTP 400 ──▶ [Error Page]
        │                                           "Invalid task ID."
        │
        ├── Task NOT FOUND ──▶ HTTP 404 ──▶ [Error Page]
        │                                   "Task not found."
        │
        └── Task FOUND ──▶ HTTP 200 ──▶ [Task Edit Form]
                                          pre-populated with current values
                                          │
                                          │  user modifies one or more fields
                                          │  click "Save Changes"
                                          │  POST /tasks/:id (_method=PUT)
                                          ▼
                                    ┌─────────────────────────────┐
                                    │  Server validates input      │
                                    └─────────────────────────────┘
                                          │
                                          ├── PASSES ──▶ DB UPDATE ──▶ HTTP 302
                                          │                             ──▶ [Task List]
                                          │                             updated values visible ✓
                                          │
                                          ├── FAILS ──▶ HTTP 422 ──▶ [Task Edit Form]
                                          │                           submitted values preserved
                                          │                           inline errors shown
                                          │
                                          └── DB ERROR ──▶ HTTP 500 ──▶ [Error Page]
                                                                         "Unable to save the task.
                                                                          Please try again."
```

### Steps

1. **Entry:** From the Task List, user clicks a task's title link or an "Edit" action button on the row. Both navigate to `GET /tasks/:id/edit`.
2. **Form Loads Pre-Populated:** All four fields (title, description, status, due date) are filled with the task's current stored values. The status dropdown shows the task's current status selected (US-2.1). User can change just the status without touching the other fields (JRN-01.3 key moment).
3. **Edit Fields:** User changes only what they need. Common case: single status update (JRN-01.3 Update Status stage).
4. **Submit:** "Save Changes" button posts to `PUT /tasks/:id` (or `POST /tasks/:id` with `_method=PUT`).
5. **Success:** HTTP 302 → Task List. Updated values are reflected immediately. `updated_at` is set to current UTC. `created_at` is not modified (US-2.2).
6. **Validation Error:** HTTP 422. Form re-renders with submitted values (not original) and inline error messages (US-2.4). All errors shown simultaneously.
7. **Not Found / Invalid ID:** Error pages at 404 and 400 respectively (US-2.1).
8. **Cancel:** A "← Cancel" link navigates back to `GET /tasks` without saving changes.

### Key UX Decisions

- **Pre-population is the critical trust signal** (JRN-01.3 Open Edit stage): seeing existing data intact reassures the user they won't accidentally wipe other fields when changing just status.
- **Status dropdown** is always selected (never blank) on the edit form — status is required for edit (unlike creation where it silently defaults).
- **Cancel link** prominently available to avoid a "trapped" feeling on a separate edit page.
- **Breadcrumb or back link** (`← Back to Tasks`) improves orientation in the edit view.
