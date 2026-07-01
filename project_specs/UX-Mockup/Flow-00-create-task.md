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
