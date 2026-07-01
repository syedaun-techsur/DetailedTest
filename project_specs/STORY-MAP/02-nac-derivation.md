---

## NaC Derivation Table

Full traceability chain for every NaC: JTBD outcome → Journey stage → NaC statement → User Story it verifies.

| NaC-ID | JTBD-ID | JTBD Outcome | Journey Stage | Natural Acceptance Criterion (NaC) | Story |
|--------|---------|-------------|--------------|-------------------------------------|-------|
| NaC-01 | JTBD-01.1 | Task captured in ≤60 seconds; creation requires no login or account setup | JRN-01.1: Navigate | "New Task" button visible on task list; Marcus reaches creation form in 1 click from the task list | US-0.1 |
| NaC-02 | JTBD-01.1 | Only title required; optional fields do not block submission; 3 clicks or fewer | JRN-01.1: Create | Submitting with title only succeeds; description, status, and due date may be omitted without error; form completes in ≤3 clicks from task list | US-0.2 |
| NaC-03 | JTBD-01.1 | Creation flow does not abandon the user on error; entered data is preserved | JRN-01.1: Create (error path) | Inline errors appear on the same form with all previously entered values intact; Marcus corrects and resubmits in one cycle | US-0.3 |
| NaC-04 | JTBD-01.1 | New task immediately visible in shared list after creation | JRN-01.1: Submit | Newly created task appears at the top of `GET /tasks` immediately after redirect (HTTP 302); visible to all team members on their next page load | US-0.1 |
| NaC-05 | JTBD-01.1 | Single click navigates from task list to task detail | JRN-01.1: Submit / JRN-01.3: Open Edit | Each task row's title is a clickable link navigating to `GET /tasks/:id` or `GET /tasks/:id/edit` in ≤1 click | US-1.2 |
| NaC-06 | JTBD-01.2 | Status of any active task readable within 10 seconds; page loads in <2 seconds | JRN-01.2: Arrive / Orient | `GET /tasks` renders within 2 seconds (≤500 tasks); each row shows title, status badge, due date, and description preview with no click required | US-1.1 |
| NaC-07 | JTBD-01.2 | Empty list is distinguishable from broken state | JRN-01.2: Orient (first visit) | When no tasks exist, message "No tasks yet. Create your first task to get started." is displayed; "New Task" button still visible; HTTP 200 returned | US-1.3 |
| NaC-08 | JTBD-01.3 | Status update persisted to DB; changes survive restart; ≤2 clicks from task list | JRN-01.3: Update Status / Save | Valid edit form submission redirects to task list showing updated values; `updated_at` is stamped to current UTC; DB write confirmed before redirect | US-2.2 |
| NaC-09 | JTBD-01.3 | Edit form pre-populated so only the changed field needs re-entry | JRN-01.3: Open Edit | Edit form at `GET /tasks/:id/edit` pre-populates all current task values; on validation error, re-renders with submitted (not original) values intact | US-2.4 |
| NaC-10 | JTBD-01.3 | Zero data loss over 30 days; tasks survive server restart | JRN-01.3: Verify | After server restart, all previously created tasks are returned on `GET /tasks`; no task data stored in application memory | US-4.1 |
| NaC-11 | JTBD-01.3 | DB write confirmed before success redirect is sent | JRN-01.3: Save | HTTP 302 redirect for create/edit/delete is not sent until DB operation is fully awaited; DB failure returns HTTP 500 with user-readable message | US-4.3 |
| NaC-12 | JTBD-01.3 / JTBD-02.3 | Single migration command sets up DB schema; no manual SQL | (Developer setup — maps to data foundation) | Running migration command creates `tasks` table with all columns and `status` CHECK constraint; switchable between PostgreSQL and SQLite via `DATABASE_URL` only | US-4.2 |
| NaC-13 | JTBD-02.1 | Verbal status update producible in ≤2 minutes by scanning filtered list | JRN-02.1: Scan / Assess | Title, status badge, and due date all visible inline on each task row; Priya reads team state without opening any individual task | US-1.1 |
| NaC-14 | JTBD-02.3 | Any field editable by any team member; no permission prompt; pre-populated | JRN-02.3: Edit Title / Update Status | Edit form pre-populates all current values; no ownership or permission check; Priya changes only the relevant field and saves | US-2.1 |
| NaC-15 | JTBD-02.3 | Status corrected on behalf of team member in ≤2 clicks | JRN-02.3: Update Status | Status dropdown on edit form offers exactly "To Do / In Progress / Done"; selecting new status and saving updates the badge immediately in the task list for all viewers | US-2.3 |
| NaC-16 | JTBD-02.3 | Delete with confirmation removes task permanently for all viewers | JRN-02.3: Delete Stale Task | Confirmation dialog names the specific task with message stating the action cannot be undone; on confirm, task is permanently removed from DB; task list refreshes immediately for all viewers | US-3.1 |
| NaC-17 | JTBD-02.3 | Confirmation step prevents accidental deletion | JRN-02.3: Delete Stale Task (cancel path) | Cancel option returns Priya to `GET /tasks` without sending a DELETE request; task remains visible and unchanged | US-3.2 |
| NaC-18 | JTBD-02.3 | Concurrent deletion produces a clear, non-confusing state | JRN-02.3: Delete Stale Task (race path) | Attempting to delete an already-deleted task returns HTTP 404 with message "Task not found. It may have already been deleted." | US-3.3 |
| NaC-19 | JTBD-02.1 | In Progress filter isolates active work in a single click | JRN-02.1: Arrive / JRN-02.2: Focus | Clicking "In Progress" filter navigates to `/tasks?status=in_progress`; list returns only matching tasks; HTTP 200 with correct filtered results | US-5.1 |
| NaC-20 | JTBD-01.2 | Active filter visually indicated; page is refresh-safe | JRN-01.2: Filter | Active filter visually distinguished from inactive options; driven by URL parameter so correct filter is shown on direct URL load (bookmarked or shared) | US-5.2 |
| NaC-21 | JTBD-02.1 | Bookmarked filter URL opens pre-filtered In Progress view instantly | JRN-02.1: Arrive / JRN-02.2: Focus | `/tasks?status=in_progress` is bookmarkable; filter state is derived entirely from URL parameter; no server session required; shared URL shows same filtered view for any viewer | US-5.3 |
| NaC-22 | JTBD-02.2 | Empty state shown clearly when filter returns no results | JRN-02.2: Scan Due Dates (empty case) | When active filter returns no tasks, descriptive message "No tasks with status '[Status Label]'. Try a different filter or create a new task." is shown; filter controls and "New Task" button remain visible | US-5.4 |

### NaC-to-Acceptance Criteria Alignment Verification

This table confirms that each NaC is supported by the corresponding UserStory's acceptance criteria.

| NaC-ID | NaC Statement (summary) | US-ID | Aligned AC Criterion |
|--------|------------------------|-------|---------------------|
| NaC-01 | "New Task" button visible on task list | US-0.1 | AC: "A 'New Task' button or link is visible on the task list page" ✅ |
| NaC-02 | Title only required; optional fields omittable | US-0.2 | AC: "A task created without a description is stored with NULL… A task created without a due date is stored with NULL" ✅ |
| NaC-03 | Inline errors; entered data preserved on validation failure | US-0.3 | AC: "The form re-renders with all previously submitted values pre-populated" ✅ |
| NaC-04 | New task at top of list; immediately visible | US-0.1 | AC: "The newly created task appears at the top of the task list (newest-first order)… immediately visible to all team members on their next page load" ✅ |
| NaC-05 | Task row title is clickable link | US-1.2 | AC: "Each task row's title is a clickable link" ✅ |
| NaC-06 | Page loads ≤2s; inline status badge | US-1.1 | AC: "Navigating to GET /tasks renders within 2 seconds… Each task row shows: title, status badge, due date, truncated description preview" ✅ |
| NaC-07 | Empty state message; New Task still visible | US-1.3 | AC: "When no tasks exist, the message 'No tasks yet. Create your first task to get started.' is displayed… The 'New Task' button or link is still visible" ✅ |
| NaC-08 | Redirect only after DB confirmed; `updated_at` stamped | US-2.2 | AC: "Submitting with valid values redirects to GET /tasks (HTTP 302)… updated_at is updated to the current UTC timestamp on every successful save" ✅ |
| NaC-09 | Edit form pre-populated; re-renders with submitted values on error | US-2.4 | AC: "The edit form re-renders with the submitted (not original) values pre-populated" ✅ |
| NaC-10 | Tasks survive server restart; no in-memory storage | US-4.1 | AC: "After a server restart, all tasks previously created are still returned on GET /tasks… Task data is never stored in application memory" ✅ |
| NaC-11 | HTTP 302 not sent until DB write confirmed | US-4.3 | AC: "The HTTP redirect (302) for create, edit, and delete is not sent until the database write/delete is fully confirmed (awaited)" ✅ |
| NaC-12 | Single migration command; switchable via DATABASE_URL | US-4.2 | AC: "A migration command is provided… The database can be switched between PostgreSQL and SQLite via DATABASE_URL environment variable only" ✅ |
| NaC-13 | Inline title, status, due date — no click needed | US-1.1 | AC: "Each task row shows: title, status badge, due date formatted as MMM DD, YYYY" ✅ |
| NaC-14 | Pre-populated edit; no permission check | US-2.1 | AC: "All form fields are pre-populated with the task's current stored values" (no permission check in v1 — by design) ✅ |
| NaC-15 | Status dropdown; badge updated immediately | US-2.3 | AC: "The status dropdown offers exactly 'To Do', 'In Progress', 'Done'… The updated status badge is immediately visible on the task list after redirect" ✅ |
| NaC-16 | Confirmation dialog names task; permanent removal | US-3.1 | AC: "Confirmation step with message: 'Are you sure you want to delete [task title]? This action cannot be undone.'… task is permanently removed from the database" ✅ |
| NaC-17 | Cancel does not delete; returns to task list | US-3.2 | AC: "Clicking Cancel does not send a delete request to the server… user is returned to GET /tasks with no changes made" ✅ |
| NaC-18 | HTTP 404 with clear message for already-deleted task | US-3.3 | AC: "If the task ID does not exist at the time of the DELETE request, the server returns HTTP 404… message reads: 'Task not found. It may have already been deleted.'" ✅ |
| NaC-19 | In Progress filter → correct URL and filtered results | US-5.1 | AC: "Clicking a filter option navigates to the corresponding URL… task list returns only tasks matching the selected status" ✅ |
| NaC-20 | Active filter visually distinguished; URL-driven | US-5.2 | AC: "The active filter option is visually distinguished… driven by the URL parameter (not client-side state), so the page is refresh-safe" ✅ |
| NaC-21 | Bookmarked filtered URL works without session | US-5.3 | AC: "The filter state is derived entirely from the URL parameter (no server-side session required)… URL can be shared and others see the same filtered view" ✅ |
| NaC-22 | Descriptive empty state with filter controls intact | US-5.4 | AC: "When a filter is active and no tasks match, message reads: 'No tasks with status [Status Label]. Try a different filter or create a new task.'… Filter controls are still rendered" ✅ |

**All 22 NaC fully aligned with UserStory acceptance criteria. 0 misalignments.**

---
