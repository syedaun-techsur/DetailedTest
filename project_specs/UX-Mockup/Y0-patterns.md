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
