# UX Mockup — TaskFlow

**Project:** TaskFlow
**Generated:** 2026-07-01
**Based on:** UserStories-TaskFlow.md v1.0, PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, JOURNEYS-TaskFlow.md v1.0, PROJECT.md v1.0

---

## Overview

TaskFlow is a minimal shared task-tracking web application for small internal teams (2–15 people). The UX is intentionally conventional: server-rendered HTML pages, standard form interactions, and no client-side routing. The product has one primary surface — the **Task List** — and two secondary surfaces: **Task Create Form** and **Task Edit Form**. Every user journey begins and ends at the Task List.

### Design Principles

1. **Task List is the hub.** Every flow returns to `/tasks`. All CRUD actions are reachable within 3 clicks from the list.
2. **Minimal friction for creation.** Only the title is required. Optional fields are visually de-emphasized so users (esp. Marcus) can capture a task in seconds without being blocked by optional fields.
3. **Pre-populated edits.** The edit form always shows current values — users change only what they need without re-entering existing data.
4. **Mandatory confirmation for deletion.** A confirmation gate (modal dialog or dedicated page) is non-negotiable before permanent deletion. The message names the task title and states "This action cannot be undone."
5. **Bookmarkable filter state.** The active status filter is encoded in the URL (`?status=in_progress`) so Priya can bookmark her morning view.
6. **All states designed.** Default, loading, success, error, and empty states are explicitly designed for every screen.
7. **WCAG AA compliance.** Status badges have sufficient color contrast; all form inputs have associated `<label>` elements; keyboard navigation works throughout.

### Screen Inventory

| Screen | Route | User Stories |
|--------|-------|-------------|
| Task List | `GET /tasks` | US-1.1, US-1.2, US-1.3, US-5.1, US-5.2, US-5.3, US-5.4 |
| Task Create Form | `GET /tasks/new` | US-0.1, US-0.2, US-0.3 |
| Task Edit Form | `GET /tasks/:id/edit` | US-2.1, US-2.2, US-2.3, US-2.4 |
| Delete Confirmation | (modal/dialog on Task List) | US-3.1, US-3.2, US-3.3 |
| Error Pages | (400, 404, 500) | US-3.3, US-4.3 |

### Flow Inventory

| Flow | Trigger | User Stories |
|------|---------|-------------|
| Flow-00: Create Task | Click "New Task" from Task List | US-0.1, US-0.2, US-0.3 |
| Flow-01: View & Filter Task List | Load `/tasks` | US-1.1, US-1.2, US-1.3, US-5.1–5.4 |
| Flow-02: Edit Task | Click task row or Edit button | US-2.1, US-2.2, US-2.3, US-2.4 |
| Flow-03: Delete Task | Click Delete button | US-3.1, US-3.2, US-3.3 |
---

## Flow 00: Create Task

**Trigger:** User clicks "New Task" button on the Task List page
**User Stories:** US-0.1, US-0.2, US-0.3
**Personas:** PER-01 Marcus Webb (primary), PER-02 Priya Nair (secondary)
**Journey Reference:** JRN-01.1 (Capturing a New Task Mid-Flow)

```
[Task List — GET /tasks]
        │
        │  click "New Task"
        ▼
[Task Create Form — GET /tasks/new]
        │
        │  fill title (required)
        │  fill description, status, due date (optional)
        │
        │  click "Save Task"
        ▼
  ┌─────────────────────────────┐
  │  Server validates input      │
  └─────────────────────────────┘
        │
        ├── Validation PASSES ──▶ DB INSERT ──▶ HTTP 302 ──▶ [Task List]
        │                                                    new task at top ✓
        │
        ├── Validation FAILS ──▶ HTTP 422 ──▶ [Task Create Form]
        │                                      form re-rendered with:
        │                                      • submitted values preserved
        │                                      • inline error messages shown
        │
        └── DB Write ERROR ──▶ HTTP 500 ──▶ [Error Page]
                                             "Unable to save the task.
                                              Please try again."
```

### Steps

1. **Entry:** User is on the Task List (`GET /tasks`). A prominent "New Task" button is visible in the page header area, above the filter controls.
2. **Navigate:** Click "New Task" → browser navigates to `GET /tasks/new`. Form renders with empty fields. Status dropdown defaults to "To Do".
3. **Fill Form:** User types a title (required). Optionally: fills in description textarea, changes status dropdown, selects a due date. Optional fields are labeled "(optional)" to reduce cognitive friction (JRN-01.1 key moment: Marcus should not feel blocked by optional fields).
4. **Submit:** User clicks "Save Task". Form posts to `POST /tasks`.
5. **Success:** Server redirects to `GET /tasks`. Newly created task appears at the top of the list (newest-first order). No explicit success toast required (list position is implicit confirmation), but a subtle success indicator is a delight opportunity (see JRN-01.1 Submit stage).
6. **Error — Validation:** Form re-renders at HTTP 422. All previously typed values are preserved. Inline error messages appear adjacent to invalid fields. Multiple errors shown simultaneously (US-0.3).
7. **Error — DB:** HTTP 500 error page rendered. User can navigate back using browser back button or a "← Back to Tasks" link on the error page.

### Key UX Decisions

- **Title field is the first and most prominent field** — tab order starts here, autofocus on page load.
- **Optional labels** on description, status, and due date reduce the fear of "do I need to fill these in?" (addresses JRN-01.1 Create stage pain point).
- **Status defaults to "To Do"** — the dropdown is pre-selected on form load; user does not need to interact with it unless changing the default.
- **Cancel link** ("← Back to Tasks") is available at all times so users can exit without submitting.
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
---

## Screen 00: Task List

**Route:** `GET /tasks` / `GET /tasks?status={slug}`
**Purpose:** Primary hub — shows all tasks, supports filtering by status, entry point to all CRUD operations.
**User Stories:** US-1.1, US-1.2, US-1.3, US-5.1, US-5.2, US-5.3, US-5.4
**Journey Reference:** JRN-01.2 (Morning Orientation), JRN-02.1 (Daily Status Review), JRN-02.2 (Triage Overdue)

---

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER                                                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  TaskFlow                          [ + New Task ]     │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  FILTER BAR                                                  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  [ All ]  [ To Do ]  [ In Progress ]  [ Done ]        │  │
│  │  (active filter visually highlighted)                 │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  TASK LIST                                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  ┌──────────────────────────────────────────────────┐ │  │
│  │  │  Fix API endpoint timeout          [In Progress] │ │  │
│  │  │  Due: Jul 15, 2026                               │ │  │
│  │  │  Investigate why /tasks endpoint times out...    │ │  │
│  │  │                           [ Edit ]  [ Delete ]   │ │  │
│  │  └──────────────────────────────────────────────────┘ │  │
│  │  ┌──────────────────────────────────────────────────┐ │  │
│  │  │  Update README with setup instructions  [To Do]  │ │  │
│  │  │  Due: —                                          │ │  │
│  │  │  (no description)                                │ │  │
│  │  │                           [ Edit ]  [ Delete ]   │ │  │
│  │  └──────────────────────────────────────────────────┘ │  │
│  │  ┌──────────────────────────────────────────────────┐ │  │
│  │  │  Deploy to staging                      [Done]   │ │  │
│  │  │  Due: Jul 01, 2026                               │ │  │
│  │  │  Deployed build #42 to staging env...            │ │  │
│  │  │                           [ Edit ]  [ Delete ]   │ │  │
│  │  └──────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Task row click targets:**
- Task **title** → `GET /tasks/:id/edit` (primary navigation)
- **[Edit]** link → `GET /tasks/:id/edit`
- **[Delete]** link/button → triggers confirmation flow

---

### Information Hierarchy

| Priority | Content | Placement | Rationale |
|----------|---------|-----------|-----------|
| Primary | Task title (clickable) | Row top-left, bold | Most important — what is the work? |
| Primary | Status badge | Row top-right | Instant scan of work state (JRN-02.1) |
| Secondary | Due date | Below title, left | Priya scans for overdue without clicking (JRN-02.2) |
| Secondary | Description preview | Below due date, muted | Context without opening task |
| Tertiary | Edit / Delete actions | Row bottom-right | Available but not dominant |
| Navigation | Filter tabs | Above list | Always visible, 1-click filter access |
| Navigation | New Task button | Header top-right | Prominent but not distracting |

---

### States

| State | Appearance | User Feedback |
|-------|------------|---------------|
| Default | List of task rows, newest first, active filter highlighted | Task rows render with title, badge, date, preview |
| Loading | Page skeleton or browser loading indicator | Server renders synchronously; no async skeleton needed for v1 |
| Empty (unfiltered) | No task rows; empty state message centered | "No tasks yet. Create your first task to get started." + New Task button visible |
| Empty (filtered) | No task rows; filter-specific empty state | "No tasks with status '[Status Label]'. Try a different filter or create a new task." + filter tabs + New Task button visible |
| DB Error (500) | Error page (separate screen) | "Unable to load tasks. Please try again." |

---

### Status Badge Design

Status badges are pill/chip elements with background color and text:

| Status | Background | Text | Accessible? |
|--------|-----------|------|-------------|
| To Do | Light gray `#E5E7EB` | Dark gray `#374151` | ✅ WCAG AA |
| In Progress | Light blue `#DBEAFE` | Dark blue `#1D4ED8` | ✅ WCAG AA |
| Done | Light green `#D1FAE5` | Dark green `#065F46` | ✅ WCAG AA |

> Note: Exact hex values are suggestions — implementers must verify contrast ratios ≥ 4.5:1 (WCAG AA for normal text).

---

### Interactive Elements

| Element | Type | Behavior | Story |
|---------|------|----------|-------|
| New Task button | Primary CTA | Navigate to `GET /tasks/new` | US-0.1 |
| Filter tabs (All / To Do / In Progress / Done) | Navigation links | Navigate to `GET /tasks?status={slug}` | US-5.1, US-5.2 |
| Task title (in each row) | Clickable link | Navigate to `GET /tasks/:id/edit` | US-1.2 |
| Edit button (per row) | Secondary action | Navigate to `GET /tasks/:id/edit` | US-2.1 |
| Delete button (per row) | Destructive action | Open confirmation dialog | US-3.1 |

---

### Empty State Designs

**Empty (Unfiltered):**
```
┌─────────────────────────────────────────────────────────────┐
│  [ All ]  [ To Do ]  [ In Progress ]  [ Done ]              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│          📋  No tasks yet.                                  │
│          Create your first task to get started.             │
│                                                             │
│                    [ + New Task ]                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Empty (Filtered — e.g., "In Progress"):**
```
┌─────────────────────────────────────────────────────────────┐
│  [ All ]  [ To Do ]  [● In Progress ]  [ Done ]             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│          No tasks with status "In Progress".                │
│          Try a different filter or create a new task.       │
│                                                             │
│                    [ + New Task ]                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
(US-1.3, US-5.4)
---

## Screen 01: Task Create Form

**Route:** `GET /tasks/new`
**Purpose:** Allow any team member to create a new task with a title and optional fields, submitted to `POST /tasks`.
**User Stories:** US-0.1, US-0.2, US-0.3
**Journey Reference:** JRN-01.1 (Capturing a New Task Mid-Flow — Marcus)

---

### Layout — Default State

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER                                                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  TaskFlow                                             │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  BREADCRUMB / BACK LINK                                      │
│  ← Back to Tasks                                            │
├─────────────────────────────────────────────────────────────┤
│  FORM — New Task                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  Title *                                              │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │                                                 │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  Description  (optional)                              │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │                                                 │ │  │
│  │  │                                                 │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  Status  (optional)                                   │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │  To Do                                        ▼ │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  Due Date  (optional)                                 │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │  YYYY-MM-DD                                     │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  [ Save Task ]               [ Cancel ]               │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

### Layout — Validation Error State (HTTP 422)

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER                                                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  TaskFlow                                             │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  ← Back to Tasks                                            │
├─────────────────────────────────────────────────────────────┤
│  FORM — New Task                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │ ⚠ Please fix the errors below before saving.   │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │                                                       │  │
│  │  Title *                                              │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │                                   ← error border│ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │  ⚠ Title is required.                                │  │
│  │                                                       │  │
│  │  Due Date  (optional)                                 │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │  2026-02-30                       ← error border│ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │  ⚠ Due date must be a valid date (YYYY-MM-DD).       │  │
│  │                                                       │  │
│  │  [ Save Task ]               [ Cancel ]               │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

### Information Hierarchy

| Priority | Content | Placement | Rationale |
|----------|---------|-----------|-----------|
| Primary | Title field | First field, top | Required; autofocused on load (US-0.1) |
| Primary | Inline error messages | Adjacent to each invalid field | Must be visible without scrolling (US-0.3) |
| Secondary | Description textarea | After title | Optional; multi-line; de-emphasized label |
| Secondary | Status dropdown | After description | Defaults to "To Do"; optional for creation |
| Secondary | Due date input | After status | Optional; date picker or text input |
| Tertiary | Optional labels | Beside field labels | Reduces anxiety about required fields (JRN-01.1) |
| Navigation | Save Task button | Bottom-left, primary | Main CTA |
| Navigation | Cancel link | Bottom-right, secondary | Exit without saving |

---

### States

| State | Appearance | User Feedback |
|-------|------------|---------------|
| Default | Empty form, status dropdown shows "To Do", autofocus on title | No messages; ready to type |
| Partially filled | Title has content; optional fields may be empty | No feedback until submit |
| Validation error (422) | Red border on invalid fields; error summary banner at top; inline messages below each invalid field | "⚠ Please fix the errors below before saving." + per-field messages; previously-typed values preserved |
| Submitting | Button shows "Saving..." (disabled state) | Prevents double-submit |
| DB Error (500) | Full error page rendered | "Unable to save the task. Please try again." |

---

### Interactive Elements

| Element | Type | Behavior | Story |
|---------|------|----------|-------|
| Title input | Text input, required | Autofocused on load; validated on submit | US-0.1, US-0.3 |
| Description textarea | Textarea, optional | Multi-line; accepts any text; NULL if empty | US-0.2 |
| Status dropdown | Select, optional | 3 options; defaults to "To Do" | US-0.2 |
| Due date input | Date input (type="date"), optional | YYYY-MM-DD; NULL if empty | US-0.2 |
| Save Task button | Primary submit | POST /tasks; disabled during submission | US-0.1 |
| Cancel link | Navigation link | GET /tasks; no data is saved | US-0.1 |

---

### Validation Error Messages (inline)

| Field | Condition | Message |
|-------|-----------|---------|
| Title | Empty or whitespace | "Title is required." |
| Title | > 255 characters | "Title must be 255 characters or fewer." |
| Due date | Invalid date format | "Due date must be a valid date (YYYY-MM-DD)." |

All errors shown simultaneously. Form re-renders with submitted values preserved (US-0.3).
---

## Screen 02: Task Edit Form

**Route:** `GET /tasks/:id/edit`
**Purpose:** Allow any team member to modify any field of an existing task. Form pre-populated with current values.
**User Stories:** US-2.1, US-2.2, US-2.3, US-2.4
**Journey Reference:** JRN-01.3 (Progressing a Status Update — Marcus), JRN-02.3 (Curating the Task List — Priya)

---

### Layout — Default State (Pre-Populated)

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER                                                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  TaskFlow                                             │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  BREADCRUMB / BACK LINK                                      │
│  ← Back to Tasks                                            │
├─────────────────────────────────────────────────────────────┤
│  FORM — Edit Task                                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  Title *                                              │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │  Fix API endpoint timeout                       │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  Description  (optional)                              │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │  Investigate why /tasks endpoint times out      │ │  │
│  │  │  when the database has > 100 rows.              │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  Status *                                             │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │  In Progress                                  ▼ │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  Due Date  (optional)                                 │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │  2026-07-15                                     │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │  [ Save Changes ]     [ Delete Task ]   [ Cancel ]    │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Note:** "Delete Task" button in the edit form is an optional convenience. If included, it triggers the same confirmation flow as the list-row Delete button (Flow-03). It may also be omitted from the edit form in favor of keeping the delete action solely on the list view.

---

### Layout — Validation Error State (HTTP 422)

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER + ← Back to Tasks                                   │
├─────────────────────────────────────────────────────────────┤
│  FORM — Edit Task                                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │ ⚠ Please fix the errors below before saving.   │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │                                                       │  │
│  │  Title *                                              │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │                                   ← error border│ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │  ⚠ Title is required.                                │  │
│  │                                                       │  │
│  │  [rest of form with submitted values preserved]       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

### Information Hierarchy

| Priority | Content | Placement | Rationale |
|----------|---------|-----------|-----------|
| Primary | Title field (pre-populated) | First field | Required; users may only need to change status |
| Primary | Status dropdown (pre-populated, required) | Third field | Most common single-field edit (JRN-01.3) |
| Primary | Inline error messages | Adjacent to invalid fields | Immediate, scannable feedback (US-2.4) |
| Secondary | Description textarea (pre-populated) | Second field | Full text preserved; scroll if long |
| Secondary | Due date input (pre-populated) | Fourth field | Optional; clearing sets NULL |
| Navigation | Save Changes button | Bottom-left, primary | Main CTA |
| Navigation | Cancel link | Bottom-right, secondary | Exit to task list |
| Navigation | Delete Task button | Bottom-center, destructive | Available as shortcut (optional placement) |

---

### States

| State | Appearance | User Feedback |
|-------|------------|---------------|
| Default (pre-populated) | All fields show current task values; status always has a value selected | Ready to edit; no messages |
| Partially edited | One or more fields changed from original | No feedback until submit |
| Not Found (404) | Error page rendered before form loads | "Task not found." |
| Invalid ID (400) | Error page rendered | "Invalid task ID." |
| Validation error (422) | Error banner + red borders + inline messages; submitted values preserved (not original values) | "⚠ Please fix the errors below before saving." |
| Submitting | "Saving..." disabled button | Prevents double-submit |
| DB Error (500) | Error page | "Unable to save the task. Please try again." |

---

### Interactive Elements

| Element | Type | Behavior | Story |
|---------|------|----------|-------|
| Title input | Text input, required | Pre-populated; validated on submit | US-2.1, US-2.4 |
| Description textarea | Textarea, optional | Pre-populated; clearing saves NULL | US-2.2 |
| Status dropdown | Select, required | Pre-selected; 3 options; cannot be blank | US-2.3 |
| Due date input | Date input, optional | Pre-populated; clearing saves NULL | US-2.2 |
| Save Changes button | Primary submit | PUT /tasks/:id; disabled during submission | US-2.2 |
| Delete Task button | Destructive action (optional on form) | Triggers delete confirmation | US-3.1 |
| Cancel link | Navigation link | GET /tasks; no data saved | US-2.1 |

---

### Validation Error Messages (inline)

| Field | Condition | Message |
|-------|-----------|---------|
| Title | Empty or whitespace after trim | "Title is required." |
| Title | > 255 characters | "Title must be 255 characters or fewer." |
| Due date | Invalid date provided | "Due date must be a valid date (YYYY-MM-DD)." |
| Status | Not one of the three valid values | "Status must be one of: To Do, In Progress, Done." |

On validation failure, re-rendered form shows **submitted values** (not the original stored values). All errors shown simultaneously (US-2.4).

---

### Differences from Create Form

| Aspect | Create Form | Edit Form |
|--------|-------------|-----------|
| Fields pre-populated | No | Yes — all 4 fields |
| Status required | No (defaults to "To Do") | Yes (always has a value) |
| Form action | POST /tasks | POST /tasks/:id (_method=PUT) |
| Page title | "New Task" | "Edit Task" |
| Cancel destination | GET /tasks | GET /tasks |
| Delete action available | No | Optional |
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
---

## Screen 04: Error Pages

**Routes:** Various (400, 404, 500 responses)
**Purpose:** Communicate error conditions clearly and provide a path back to the working application.
**User Stories:** US-2.1, US-3.3, US-4.3
**Feature Refs:** FRD Y2 (Cross-Feature Error Catalog)

---

### Layout — Generic Error Page Pattern

All error pages share a consistent layout:

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER                                                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  TaskFlow                                             │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│     [Error Icon or Code]                                    │
│                                                             │
│     [Error Heading]                                         │
│     [Error Message]                                         │
│                                                             │
│               [ ← Back to Task List ]                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Error Page Variants

#### 400 — Invalid Task ID

```
400
Invalid Task ID
"The task ID provided is not valid."
[ ← Back to Task List ]
```

**Trigger:** `:id` path parameter is not a positive integer (e.g., `/tasks/abc`, `/tasks/-1`).
**User Stories:** US-2.1 (edit), US-3.3 (delete)

---

#### 404 — Task Not Found

```
404
Task Not Found
"Task not found."                         (from edit — US-2.1)
   — or —
"Task not found. It may have already      (from delete — US-3.3)
 been deleted."
[ ← Back to Task List ]
```

**Trigger:** `:id` references a row that does not exist in the `tasks` table.
**Contextual message:** Edit context uses "Task not found."; Delete context uses "Task not found. It may have already been deleted." The more informative message helps Priya understand that a task was likely already removed by a colleague.

---

#### 500 — Server / Database Error

```
500
Something Went Wrong
"Unable to load tasks. Please try again."       (list read failure)
   — or —
"Unable to save the task. Please try again."    (create/edit write failure)
   — or —
"Unable to delete the task. Please try again."  (delete failure)
[ ← Back to Task List ]
```

**Trigger:** Database connection failure, write failure, read failure, or schema missing.
**User Stories:** US-4.3 (users must not receive success redirect before confirmed DB write)

---

### States

| Error | HTTP Status | Message | Recovery Path |
|-------|------------|---------|---------------|
| Invalid task ID | 400 | "The task ID provided is not valid." | Back to Task List |
| Task not found (edit) | 404 | "Task not found." | Back to Task List |
| Task not found (delete) | 404 | "Task not found. It may have already been deleted." | Back to Task List |
| DB read failure | 500 | "Unable to load tasks. Please try again." | Back to Task List |
| DB write failure | 500 | "Unable to save the task. Please try again." | Back to Task List |
| DB delete failure | 500 | "Unable to delete the task. Please try again." | Back to Task List |
| DB connection failure | 503 | "The application is temporarily unavailable. Please try again later." | Retry / contact admin |

---

### Interactive Elements

| Element | Type | Behavior |
|---------|------|----------|
| "← Back to Task List" link | Navigation link | GET /tasks |
| Page title (browser tab) | Metadata | Shows error code + app name: "404 — TaskFlow" |
---

## Interaction Patterns

### Pattern 1: Form Submission with Validation

**When to use:** Task Create (POST /tasks) and Task Edit (PUT /tasks/:id).
**Behavior:**
1. User fills form and clicks primary submit button.
2. Button transitions to disabled state with label "Saving..." (prevents double-submit).
3. Server processes the request:
   - **Success (302):** Browser follows redirect to Task List. Task appears at top (create) or with updated values (edit).
   - **Validation Failure (422):** Form re-renders. Error summary banner appears at top of form. Each invalid field gets a red border + inline error message below it. All previously-typed values are preserved. Submit button re-enables.
   - **Server Error (500):** Error page rendered. "← Back to Tasks" link provided.
4. **Multiple errors:** All validation errors are displayed simultaneously (US-0.3, US-2.4). The page scrolls to the error summary banner if the user has scrolled down.

**Examples:** Screen-01 (Create), Screen-02 (Edit)

---

### Pattern 2: Destructive Action with Confirmation Gate

**When to use:** Task Deletion (DELETE /tasks/:id).
**Behavior:**
1. User clicks "Delete" on a task row or edit form.
2. Confirmation dialog/page appears immediately. No server request has been made yet.
3. Dialog shows task title + permanent deletion warning.
4. **Cancel path:** User dismisses (click Cancel, press Escape, or click overlay). No request sent. User returns to their prior view.
5. **Confirm path:** User clicks "Delete Task". Button transitions to "Deleting..." (disabled). Server delete request is sent.
6. **Success (302):** Task List renders without the deleted task.
7. **Not Found (404):** Error page — task already deleted (concurrent deletion by teammate).
8. **Server Error (500):** Error page — task NOT deleted; user can retry.

**Cancel is always the safe/default action** — keyboard focus defaults to Cancel, not Delete.

**Examples:** Screen-03 (Delete Confirmation), Flow-03

---

### Pattern 3: URL-Driven Filter State

**When to use:** Status filtering on the Task List (F5).
**Behavior:**
1. Filter controls (4 tab-style links) are always rendered above the task list.
2. Each filter is a plain `<a href>` link — no JavaScript required for filtering.
3. The active filter is determined server-side by reading `?status=` from the URL.
4. Active filter tab receives a distinct visual style (e.g., filled background, bold text, or bottom border).
5. The URL always reflects the active filter, making every filtered view bookmarkable and shareable (US-5.3).
6. Invalid `?status=` values are silently ignored; "All" is shown as active (US-5.4).

**Pattern benefits:**
- No client-side state management needed
- Refresh-safe (US-5.2)
- Bookmarkable for Priya's daily ritual (JRN-02.1 Arrive stage)
- Zero JavaScript required for core filter functionality

**Examples:** Screen-00 (Task List), Flow-01

---

### Pattern 4: Pre-Populated Edit Form

**When to use:** Task Edit (GET /tasks/:id/edit).
**Behavior:**
1. Server reads task from DB before rendering the form.
2. Every form field is set to the task's current stored value:
   - `title` input: `value="[current title]"`
   - `description` textarea: body text = current description (or empty)
   - `status` select: `selected` attribute on the matching `<option>`
   - `due_date` input: `value="[YYYY-MM-DD]"` or empty if NULL
3. User changes only what they need — untouched fields retain their values on submit.
4. On validation failure (422): form re-renders with **submitted values** (not original DB values).

**Critical trust signal** (JRN-01.3): seeing pre-populated data intact tells the user their prior work is safe. This is the key moment where edit feels "lightweight" vs. "require re-entering everything."

---

### Pattern 5: Empty State Communication

**When to use:** Task List when zero tasks match the current view.
**Behavior:**
1. Server detects zero results from the DB query.
2. Instead of an empty list container (which would look broken), a centered empty state message is shown.
3. Message is context-sensitive:
   - Unfiltered (no tasks exist): "No tasks yet. Create your first task to get started."
   - Filtered (filter active, no matches): "No tasks with status '[Status Label]'. Try a different filter or create a new task."
4. Filter controls remain visible in all empty states (user can switch filters).
5. "New Task" button remains visible in all empty states (user can create a task to populate the list).

**Examples:** Screen-00 (Task List empty states), US-1.3, US-5.4

---

### Pattern 6: Inline Error Placement

**When to use:** Any form with server-side validation errors.
**Behavior:**
1. **Error summary banner** at the top of the form body (not page header): "⚠ Please fix the errors below before saving." This ensures keyboard/screen reader users encounter the error context first.
2. **Field-level errors** appear immediately below the invalid input, in red text, with a warning icon prefix (⚠).
3. **Invalid fields** receive a red border or `aria-invalid="true"` attribute.
4. **Error messages** are associated with their fields via `aria-describedby` for screen reader accessibility.
5. All errors are shown simultaneously — no sequential "first error only" pattern (US-0.3, US-2.4).

**Error message text** matches FRD Y2 exactly:
- `TITLE_REQUIRED` → "Title is required."
- `TITLE_TOO_LONG` → "Title must be 255 characters or fewer."
- `DUE_DATE_INVALID` → "Due date must be a valid date (YYYY-MM-DD)."
- `STATUS_INVALID` → "Status must be one of: To Do, In Progress, Done."
---

## Responsive Considerations

TaskFlow is an **internal web tool** used primarily on desktop browsers by team members at their workstations. The PRD explicitly states "web-first; browser is sufficient for internal tooling." Mobile native app is out of scope. However, basic responsive behavior ensures the app is usable on a tablet or laptop at varying window sizes.

---

### Breakpoints

| Breakpoint | Width | Target |
|-----------|-------|--------|
| Desktop | > 1024px | Primary: team members at workstations |
| Tablet | 768px – 1024px | Secondary: laptops, iPad landscape |
| Mobile | < 768px | Tertiary: mobile browser (functional but not optimized) |

---

### Desktop (> 1024px)

**Task List:**
- Full-width layout with generous padding.
- Task rows display all columns side-by-side: title (links), status badge, due date, description preview, action buttons.
- Filter tabs in a single horizontal row above the list.
- "New Task" button in the top-right of the header.

```
┌─────────────────────────────────────────────────────────────────┐
│  TaskFlow                                    [ + New Task ]     │
├─────────────────────────────────────────────────────────────────┤
│  [ All ]  [ To Do ]  [ In Progress ]  [ Done ]                  │
├─────────────────────────────────────────────────────────────────┤
│  Fix API endpoint timeout    [In Progress]  Jul 15  Investigate │
│  Update README               [To Do]        —       (no desc)   │
│  Deploy to staging           [Done]         Jul 01  Deployed #42│
└─────────────────────────────────────────────────────────────────┘
```

**Create / Edit Forms:**
- Centered single-column form, max-width ~600px.
- Label above input pattern (not inline labels) for clarity.
- All form fields visible without scrolling on typical 768px+ viewport height.

---

### Tablet (768px – 1024px)

**Task List:**
- Same layout as desktop; may require minor padding reduction.
- Action buttons (Edit, Delete) may stack vertically per row if horizontal space is tight.
- Filter tabs still in a single row — 4 short labels fit comfortably.

**Create / Edit Forms:**
- No layout change from desktop; single-column form is naturally responsive.
- Touch targets for dropdowns and date pickers should be ≥ 44px height (WCAG 2.1 touch target guideline).

---

### Mobile (< 768px)

**Task List:**
- Stack layout: title on its own line, status badge and due date on the next line, description preview below.
- Action buttons (Edit, Delete) either stack below description or collapse into a "..." overflow menu.
- Filter tabs may wrap to two rows or become a dropdown select if all 4 don't fit.
- "New Task" button may move below the filter row if header space is constrained.

```
┌──────────────────────────────────────────┐
│  TaskFlow            [ + New Task ]      │
├──────────────────────────────────────────┤
│  [All] [To Do] [In Progress] [Done]      │
├──────────────────────────────────────────┤
│  Fix API endpoint timeout                │
│  [In Progress]  •  Jul 15, 2026          │
│  Investigate why /tasks endpoint...      │
│                       [Edit] [Delete]    │
├──────────────────────────────────────────┤
│  Update README                           │
│  [To Do]  •  —                           │
│                       [Edit] [Delete]    │
└──────────────────────────────────────────┘
```

**Create / Edit Forms:**
- Full-width inputs (100% width).
- Adequate spacing between fields for touch interaction.
- Date input uses native mobile date picker (browser-native `type="date"`).

---

### Key Responsive Principles

1. **No horizontal scrolling** at any breakpoint — all content adapts to available width.
2. **Filter tabs** are the most space-constrained element at mobile — consider a `<select>` dropdown fallback at < 480px if labels overflow.
3. **Touch targets** for buttons and links: minimum 44×44px per WCAG 2.1 Success Criterion 2.5.5 (Level AAA) and iOS/Android guidelines.
4. **Font sizes** remain readable at all breakpoints — base font ≥ 16px on mobile to prevent iOS auto-zoom on inputs.
5. **Status badges** scale with text — pill shape with padding remains visible at all sizes.
---

## Accessibility Notes

TaskFlow targets **WCAG 2.1 Level AA** compliance. This is specified in the FRD non-functional constraints ("status badges meet WCAG AA color contrast") and PRD ("Form inputs have labels; status badges have sufficient color contrast (WCAG AA minimum)").

---

### Color Contrast

All text must meet WCAG AA minimum contrast ratios:
- **Normal text (< 18pt / < 14pt bold):** minimum 4.5:1
- **Large text (≥ 18pt / ≥ 14pt bold):** minimum 3:1
- **UI components and graphical objects:** minimum 3:1

**Status Badge Requirements (critical per FRD F01):**

| Badge | Background | Text | Minimum Ratio | Must verify |
|-------|-----------|------|---------------|-------------|
| To Do | `#E5E7EB` (light gray) | `#374151` (dark gray) | 4.5:1 | ✅ ~8.6:1 |
| In Progress | `#DBEAFE` (light blue) | `#1D4ED8` (dark blue) | 4.5:1 | ✅ ~5.9:1 |
| Done | `#D1FAE5` (light green) | `#065F46` (dark green) | 4.5:1 | ✅ ~7.7:1 |

> These are reference values. Implementation team must run final contrast checks using a tool such as [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) before shipping.

**Do not rely on color alone** to convey status — the badge text ("To Do", "In Progress", "Done") also communicates the status for colorblind users.

---

### Form Inputs and Labels

Every form input must have an associated `<label>` element. This is a hard requirement from the FRD (F00, F02 §Accessibility).

**Correct pattern:**
```html
<label for="title">Title <span aria-hidden="true">*</span>
  <span class="sr-only">(required)</span>
</label>
<input type="text" id="title" name="title" required aria-required="true">
```

**Optional fields:**
```html
<label for="description">
  Description <span class="visually-hidden">(optional)</span>
</label>
<textarea id="description" name="description"></textarea>
```

**Rules:**
- No `placeholder`-only labeling (placeholder text disappears on focus and has insufficient contrast).
- Required fields: use `required` attribute + `aria-required="true"`.
- Optional fields: "(optional)" in the label text.

---

### Validation Error Accessibility

**When validation errors occur (422 form re-render):**

1. **Error summary** at top of form with `role="alert"` so screen readers announce it immediately:
   ```html
   <div role="alert" class="error-summary">
     Please fix the errors below before saving.
   </div>
   ```

2. **Field-level errors** linked via `aria-describedby`:
   ```html
   <input id="title" aria-invalid="true" aria-describedby="title-error">
   <span id="title-error" class="error-message">Title is required.</span>
   ```

3. **Page focus** on form re-render: move focus to the error summary element so keyboard users immediately know an error occurred.

---

### Keyboard Navigation

All interactive elements must be reachable and operable via keyboard alone:

| Element | Keyboard Behavior |
|---------|-----------------|
| Navigation links (filter tabs, "New Task", "← Back") | Tab to focus; Enter to activate |
| Form inputs | Tab order: Title → Description → Status → Due Date → Submit → Cancel |
| Submit buttons | Enter key activates focused button |
| Delete Confirmation dialog | Escape closes dialog (Cancel path); Tab cycles between Cancel and Delete Task buttons; Enter activates focused button |
| Delete confirmation default focus | **Cancel button** receives focus when dialog opens (prevents accidental Enter-to-delete) |
| Filter tabs | Tab to navigate between tabs; Enter to activate |

**Focus management for dialog:**
- When the delete confirmation dialog opens, focus is moved into the dialog (Cancel button).
- When the dialog closes (cancel), focus returns to the "Delete" trigger element on the task list.
- Focus must not escape the dialog while it is open (focus trap).

---

### Screen Reader Considerations

**Task List:**
- Each task row should use semantic list markup (`<ul>` / `<li>`) or a `<table>` with proper headers — not `<div>` soup.
- If using a `<table>`: `<th scope="col">` for column headers (Title, Status, Due Date, Actions).
- Status badges: plain text inside the badge is sufficient; no additional ARIA needed if text is present.
- "Edit" and "Delete" action links in each row must include the task title in their accessible label to distinguish them:
  ```html
  <a href="/tasks/42/edit" aria-label="Edit Fix API endpoint timeout">Edit</a>
  <button aria-label="Delete Fix API endpoint timeout">Delete</button>
  ```
  Without this, a screen reader user hears "Edit, Edit, Edit, Delete, Delete, Delete" with no context.

**Empty States:**
- Empty state message should be in a `<p>` inside the list container (or `<td colspan>` if using a table) so it's in the correct document flow.

**Filter Tabs:**
- Use `<nav aria-label="Filter tasks by status">` wrapper.
- Active tab: `aria-current="page"` on the active `<a>` element.

**Page Titles:**
- Unique `<title>` for each page:
  - Task List: "Tasks — TaskFlow"
  - New Task: "New Task — TaskFlow"
  - Edit Task: "Edit: [task title] — TaskFlow"
  - Error: "404 Not Found — TaskFlow"

---

### ARIA Landmark Roles

Use semantic HTML or ARIA landmarks for screen reader navigation:

| Region | Element / Role |
|--------|---------------|
| App header + "New Task" button | `<header>` |
| Filter nav | `<nav aria-label="Filter tasks by status">` |
| Task list | `<main>` wrapping the list content |
| Form | `<main>` > `<form>` |
| Footer (if present) | `<footer>` |
| Delete dialog | `<dialog>` element with `aria-labelledby` pointing to dialog heading |

---

### Summary Checklist

- [ ] All status badge text/background combinations meet ≥ 4.5:1 contrast ratio
- [ ] Every form input has an associated `<label>` element
- [ ] Required fields marked with `required` + `aria-required="true"`
- [ ] Validation errors use `role="alert"` for summary and `aria-describedby` for field linkage
- [ ] `aria-invalid="true"` set on invalid fields
- [ ] Delete confirmation focuses Cancel button on open; traps focus in dialog
- [ ] Row-level Edit/Delete links include task title in `aria-label`
- [ ] Filter tabs use `aria-current="page"` for active tab
- [ ] Page `<title>` is unique per screen
- [ ] Keyboard-navigable in full Tab order without mouse
- [ ] No color-only status communication (badge text always present)
- [ ] Touch targets ≥ 44px on mobile
