---

## Screen 03: Delete Confirmation

**Route:** Triggered from task list row or edit form (modal overlay or dedicated page)
**Purpose:** Mandatory confirmation gate before permanent task deletion. Prevents accidental deletion.
**User Stories:** US-3.1, US-3.2, US-3.3
**Journey Reference:** JRN-02.3 (Delete Stale Task stage — Priya)

---

### Layout — Modal / Dialog Overlay

```
┌─────────────────────────────────────────────────────────────┐
│  [Task List page rendered behind dimmed overlay]            │
│                                                             │
│         ┌──────────────────────────────────────┐           │
│         │                                      │           │
│         │   Delete Task                        │           │
│         │   ─────────────────────────────────  │           │
│         │   Are you sure you want to delete    │           │
│         │   "Fix API endpoint timeout"?         │           │
│         │                                      │           │
│         │   This action cannot be undone.      │           │
│         │                                      │           │
│         │   [ Cancel ]     [ Delete Task ]     │           │
│         │                                      │           │
│         └──────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Button order:** Cancel on the left (primary visual focus), Delete on the right (destructive action). Keyboard: `Escape` dismisses; `Enter` focuses Cancel by default (not Delete) to prevent accidental keyboard-triggered deletion.

---

### Layout — Fallback: Dedicated Confirmation Page

For no-JavaScript environments, a server-rendered confirmation page is used:

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER                                                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  TaskFlow                                             │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  ← Back to Tasks                                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Confirm Deletion                                           │
│  ────────────────────────────────────────────              │
│                                                             │
│  Are you sure you want to delete                            │
│  "Fix API endpoint timeout"?                                │
│                                                             │
│  This action cannot be undone.                              │
│                                                             │
│  [ Cancel ]                    [ Delete Task ]              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Cancel → `GET /tasks` (no delete request sent)
Delete Task → `POST /tasks/:id/delete`

---

### Information Hierarchy

| Priority | Content | Placement | Rationale |
|----------|---------|-----------|-----------|
| Primary | Task title (interpolated) | Dialog body | User must know exactly what they are deleting (JRN-02.3 key moment) |
| Primary | "This action cannot be undone." | Below task title | Communicates permanence; no soft delete in v1 |
| Primary | Cancel button | Bottom-left, default focus | Safe default action |
| Secondary | Delete Task button | Bottom-right, destructive styling | Requires deliberate action |
| Tertiary | Dialog header "Delete Task" | Dialog top | Context label |

---

### States

| State | Appearance | User Feedback |
|-------|------------|---------------|
| Default | Dialog/page renders with task title | User reads confirmation text |
| User cancels | Dialog closes / redirect to GET /tasks | No change to task; no request sent (US-3.2) |
| User confirms | Submit button shows "Deleting..." | Brief loading state while delete request processes |
| Delete success | Redirect to GET /tasks; task removed from list | Task no longer visible |
| Task not found (404) | Error page rendered | "Task not found. It may have already been deleted." (US-3.3) |
| DB Error (500) | Error page rendered | "Unable to delete the task. Please try again." |

---

### Interactive Elements

| Element | Type | Behavior | Story |
|---------|------|----------|-------|
| Cancel button | Secondary action | Dismiss dialog; GET /tasks; NO delete request | US-3.2 |
| Delete Task button | Destructive primary | POST /tasks/:id/delete; disabled on click | US-3.1 |
| Escape key (modal) | Keyboard shortcut | Same as Cancel | US-3.2 |
| Background overlay click | Dismiss gesture | Same as Cancel | US-3.2 |

---

### Confirmation Text Template

The confirmation message must use this exact structure:

> "Are you sure you want to delete **'[task title]'**? This action cannot be undone."

Where `[task title]` is the full title of the task being deleted, HTML-escaped to prevent XSS injection.

Example:
> "Are you sure you want to delete 'Fix API endpoint timeout'? This action cannot be undone."
