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
