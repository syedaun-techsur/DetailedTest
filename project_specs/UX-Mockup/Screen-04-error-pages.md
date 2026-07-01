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
