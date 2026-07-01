---

## Flow 03: Delete Task

**Trigger:** User clicks "Delete" button on a Task List row or Task Edit form
**User Stories:** US-3.1, US-3.2, US-3.3
**Personas:** PER-01 Marcus Webb (delete own tasks — US-3.1), PER-02 Priya Nair (curate stale tasks — JRN-02.3, US-3.2)

```
[Task List — GET /tasks]
  (or Task Edit Form — GET /tasks/:id/edit)
        │
        │  click "Delete" button/link on task row
        ▼
[Delete Confirmation Dialog / Page]
  ┌──────────────────────────────────────────────────────────┐
  │  "Are you sure you want to delete '[task title]'?         │
  │   This action cannot be undone."                          │
  │                                                           │
  │   [ Confirm Delete ]      [ Cancel ]                      │
  └──────────────────────────────────────────────────────────┘
        │
        ├── User clicks CANCEL ──▶ dialog closes / redirect to GET /tasks
        │                          NO request sent to server
        │                          task unchanged ✓ (US-3.2)
        │
        └── User clicks CONFIRM ──▶ POST /tasks/:id/delete
                                    (or DELETE /tasks/:id)
                                          │
                                    ┌─────────────────────────┐
                                    │  Server looks up task    │
                                    └─────────────────────────┘
                                          │
                                          ├── :id INVALID ──▶ HTTP 400
                                          │                   [Error Page]
                                          │                   "Invalid task ID."
                                          │
                                          ├── Task NOT FOUND ──▶ HTTP 404
                                          │                       [Error Page]
                                          │                       "Task not found.
                                          │                        It may have already
                                          │                        been deleted."
                                          │                       (US-3.3)
                                          │
                                          ├── DB DELETE OK ──▶ HTTP 302
                                          │                    ──▶ [Task List]
                                          │                    task gone from list ✓
                                          │
                                          └── DB DELETE ERROR ──▶ HTTP 500
                                                                   [Error Page]
                                                                   "Unable to delete the task.
                                                                    Please try again."
```

### Steps

1. **Entry:** User clicks "Delete" on a task row (Task List) or on the Task Edit form. The "Delete" trigger is always available from the list view.
2. **Confirmation Gate:** A confirmation dialog (or dedicated confirmation page) appears with the exact task title interpolated: _"Are you sure you want to delete '[task title]'? This action cannot be undone."_ (US-3.1, FRD F03 §Process Step 2).
3. **Cancel Path:** User clicks "Cancel". Dialog closes. No HTTP request is made. User remains on the Task List with the task intact (US-3.2).
4. **Confirm Path:** User clicks "Confirm Delete". Client sends `POST /tasks/:id/delete` (HTML method override for `DELETE`).
5. **Success:** HTTP 302 → Task List. The deleted task no longer appears. Immediately visible to all viewers.
6. **Not Found (404):** Task was already deleted (e.g., by a concurrent team member). Error page with: "Task not found. It may have already been deleted." (US-3.3).
7. **DB Error (500):** Error page with retry message. Task is NOT deleted.

### Confirmation Dialog Design

The confirmation must be one of:
- **Browser `confirm()` dialog** — simplest implementation, but limited styling control.
- **Inline HTML modal/overlay** — styled, branded, allows custom button labels. Preferred for UX quality.
- **Dedicated confirmation page** (`GET /tasks/:id/delete`) — fully server-rendered, no JS required; less smooth UX.

**Recommended:** Inline HTML modal (e.g., `<dialog>` element or simple overlay div). Falls back to confirmation page for no-JS environments.

### Key UX Decisions

- **Task title in confirmation message** is mandatory — "Are you sure you want to delete this task?" (without the title) is ambiguous and creates risk (JRN-02.3 Delete Stale Task key moment).
- **"This action cannot be undone"** text must appear to communicate permanence (no soft-delete in v1).
- **Cancel is the default-focus button** — keyboard users pressing Enter should cancel, not confirm, to prevent accidental deletion.
- **"Delete" button on the task list row** should be visually distinct from "Edit" (e.g., red or destructive styling) but not so alarming that users avoid using it for legitimate cleanup.
