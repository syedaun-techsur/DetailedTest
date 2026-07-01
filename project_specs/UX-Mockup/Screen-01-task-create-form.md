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
