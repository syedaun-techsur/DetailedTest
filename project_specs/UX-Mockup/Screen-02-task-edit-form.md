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
