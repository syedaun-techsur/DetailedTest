# User Stories — TaskFlow
**Project Acronym:** TaskFlow
**Version:** 1.0
**Date:** 2026-07-01
**Status:** Draft
**Based On:** PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, PERSONAS-TaskFlow.md v1.0

---

## Personas

| ID | Name | Role |
|----|------|------|
| PER-01 | Marcus Webb | Team Member (Individual Contributor) |
| PER-02 | Priya Nair | Team Lead / Coordinator |

---

## Priority Definitions

| Priority | Label | Description |
|----------|-------|-------------|
| P0 | Critical | Must ship for MVP; application is non-functional without it |
| P1 | High | Ships with MVP; significant usability impact if absent |
| P2 | Medium | Post-MVP; valuable but not blocking initial release |
| P3 | Low | Nice-to-have; deferred to v2+ backlog |

---

## Epic 0: Task Creation (F0)

*Covers the ability for any team member to create a new task in the shared workspace via a form, with immediate database persistence and redirect to the task list.*

---

### US-0.1: Create a New Task with a Title
**As a** Marcus Webb (team member), **I want to** submit a task creation form with a title, **so that** my work item is immediately logged in the shared team task list.

**Acceptance Criteria:**
- [ ] A "New Task" button or link is visible on the task list page
- [ ] Navigating to the creation form renders a page at `GET /tasks/new` with a title field, description textarea, status dropdown, and due date input
- [ ] The status dropdown defaults to "To Do" when the form loads
- [ ] Submitting the form with a valid title redirects to `GET /tasks` (HTTP 302)
- [ ] The newly created task appears at the top of the task list (newest-first order)
- [ ] The new task is immediately visible to all team members on their next page load

**Priority:** P0 | **Feature Ref:** F0

---

### US-0.2: Create a Task with Optional Fields
**As a** Marcus Webb (team member), **I want to** optionally add a description, set a status, and specify a due date when creating a task, **so that** I can provide full context for a work item in a single step.

**Acceptance Criteria:**
- [ ] Description field is a multi-line textarea and accepts any text input
- [ ] Status dropdown offers exactly three options: "To Do", "In Progress", "Done"
- [ ] Due date field accepts a date in `YYYY-MM-DD` format via a date picker
- [ ] A task created without a description is stored with a NULL description (no error shown)
- [ ] A task created without a due date is stored with a NULL due date (no error shown)
- [ ] A task created without explicitly choosing a status defaults to "To Do"

**Priority:** P0 | **Feature Ref:** F0

---

### US-0.3: See Validation Errors on Invalid Task Creation
**As a** Marcus Webb (team member), **I want to** see clear inline error messages when I submit an incomplete or invalid creation form, **so that** I can correct my input and resubmit without losing what I already typed.

**Acceptance Criteria:**
- [ ] Submitting the form with an empty or whitespace-only title shows the error: "Title is required."
- [ ] Submitting the form with a title exceeding 255 characters shows: "Title must be 255 characters or fewer."
- [ ] Submitting an invalid due date (e.g., `2026-02-30`) shows: "Due date must be a valid date (YYYY-MM-DD)."
- [ ] The form re-renders with all previously submitted values pre-populated (user does not lose their input)
- [ ] Multiple validation errors are displayed simultaneously (not just the first one)
- [ ] The server returns HTTP 422 on validation failure

**Priority:** P0 | **Feature Ref:** F0

---

## Epic 1: Task List View (F1)

*Covers the primary shared task list — the hub from which all team members monitor work status, navigate to individual tasks, and initiate CRUD operations.*

---

### US-1.1: View All Tasks at a Glance
**As a** Priya Nair (team lead), **I want to** open the task list and see all tasks in the shared workspace at once, **so that** I can get an accurate snapshot of team workload without asking team members for status updates.

**Acceptance Criteria:**
- [ ] Navigating to `GET /tasks` renders the task list page within 2 seconds (up to 500 tasks)
- [ ] Each task row shows: title, status badge, due date formatted as `MMM DD, YYYY` (blank if no due date), and a truncated description preview (first 100 characters + `…` if longer)
- [ ] Tasks are sorted by creation date, newest first
- [ ] The page returns HTTP 200 on success
- [ ] A "New Task" button or link is visible on the page

**Priority:** P0 | **Feature Ref:** F1

---

### US-1.2: Navigate from the Task List to a Task's Detail or Edit View
**As a** Marcus Webb (team member), **I want to** click on a task in the list to open its full detail or edit view, **so that** I can review or update it in 3 clicks or fewer from the task list.

**Acceptance Criteria:**
- [ ] Each task row's title is a clickable link
- [ ] Clicking a task row navigates to `GET /tasks/:id/edit` (the pre-populated edit form serves as the task detail view)
- [ ] The linked page shows the full task details (title, description, status, due date) pre-populated in the edit form
- [ ] The navigation completes without error for any valid task ID

**Priority:** P0 | **Feature Ref:** F1

---

### US-1.3: See an Empty State When No Tasks Exist
**As a** Marcus Webb (team member), **I want to** see a helpful message when there are no tasks in the workspace, **so that** I understand the list is empty (not broken) and know how to add the first task.

**Acceptance Criteria:**
- [ ] When no tasks exist (unfiltered), the message "No tasks yet. Create your first task to get started." is displayed
- [ ] The "New Task" button or link is still visible in the empty state
- [ ] The page returns HTTP 200 (not an error page)
- [ ] No blank or broken list container is shown

**Priority:** P0 | **Feature Ref:** F1

---

## Epic 2: Task Edit (F2)

*Covers the ability for any team member to modify any field of an existing task and persist the changes, with the same validation rules as task creation.*

---

### US-2.1: Open the Edit Form Pre-Populated with Current Task Values
**As a** Priya Nair (team lead), **I want to** open an edit form that shows the current values of a task, **so that** I can change only the fields that need updating without re-entering everything.

**Acceptance Criteria:**
- [ ] Navigating to `GET /tasks/:id/edit` renders an edit form
- [ ] All form fields (title, description, status, due date) are pre-populated with the task's current stored values
- [ ] The status dropdown has the task's current status selected
- [ ] If a task ID does not exist, the server returns HTTP 404 with the message "Task not found."
- [ ] If the task ID is not a positive integer (e.g., `/tasks/abc`), the server returns HTTP 400 with "Invalid task ID."

**Priority:** P0 | **Feature Ref:** F2

---

### US-2.2: Save Changes to a Task
**As a** Marcus Webb (team member), **I want to** edit a task's title, description, status, or due date and save the changes, **so that** the task reflects current reality for everyone on the team.

**Acceptance Criteria:**
- [ ] Submitting the edit form with valid values redirects to `GET /tasks` (HTTP 302)
- [ ] The task list shows the updated values immediately after redirect
- [ ] `updated_at` is updated to the current UTC timestamp on every successful save
- [ ] `created_at` is never modified by an edit operation
- [ ] Clearing the description field saves NULL (no error)
- [ ] Clearing the due date field saves NULL (no error)

**Priority:** P0 | **Feature Ref:** F2

---

### US-2.3: Update a Task's Status
**As a** Priya Nair (team lead), **I want to** change a task's status to "In Progress" or "Done" on behalf of a team member, **so that** the shared task list accurately reflects current work state without requiring that team member to be online.

**Acceptance Criteria:**
- [ ] The status dropdown on the edit form offers exactly: "To Do", "In Progress", "Done"
- [ ] Selecting a new status and saving updates the task's status in the database
- [ ] The updated status badge is immediately visible on the task list after redirect
- [ ] Submitting the form with a blank or invalid status returns HTTP 422 with: "Status must be one of: To Do, In Progress, Done."

**Priority:** P0 | **Feature Ref:** F2

---

### US-2.4: See Validation Errors on Invalid Task Edit
**As a** Marcus Webb (team member), **I want to** see inline errors when I submit invalid edits, **so that** I know exactly what to fix and my other input is preserved.

**Acceptance Criteria:**
- [ ] Submitting an empty title shows: "Title is required."
- [ ] Submitting a title over 255 characters shows: "Title must be 255 characters or fewer."
- [ ] Submitting an invalid due date shows: "Due date must be a valid date (YYYY-MM-DD)."
- [ ] The edit form re-renders with the submitted (not original) values pre-populated
- [ ] All validation errors are shown simultaneously
- [ ] Server returns HTTP 422 on validation failure

**Priority:** P0 | **Feature Ref:** F2

---

## Epic 3: Task Deletion (F3)

*Covers permanent deletion of a task from the shared workspace, with a mandatory confirmation step to prevent accidental loss.*

---

### US-3.1: Delete a Task with a Confirmation Step
**As a** Marcus Webb (team member), **I want to** delete a task I created that is no longer relevant, with a confirmation prompt before the deletion happens, **so that** I don't accidentally remove tasks that other team members are still relying on.

**Acceptance Criteria:**
- [ ] A "Delete" button or link is available on the task list row and/or task detail view
- [ ] Clicking "Delete" presents a confirmation step with the message: "Are you sure you want to delete '[task title]'? This action cannot be undone."
- [ ] The user must take an affirmative action (confirm) before the deletion is executed
- [ ] On confirmation, the task is permanently removed from the database
- [ ] After deletion, the server redirects to `GET /tasks` (HTTP 302)
- [ ] The deleted task no longer appears in the task list for any viewer

**Priority:** P0 | **Feature Ref:** F3

---

### US-3.2: Cancel a Deletion and Return to the Task List
**As a** Priya Nair (team lead), **I want to** cancel a deletion action if I clicked Delete by mistake, **so that** no task is accidentally removed.

**Acceptance Criteria:**
- [ ] The confirmation step includes a "Cancel" option (link, button, or dialog dismiss)
- [ ] Clicking Cancel does not send a delete request to the server
- [ ] The user is returned to `GET /tasks` with no changes made
- [ ] The task remains visible and unchanged in the task list

**Priority:** P0 | **Feature Ref:** F3

---

### US-3.3: Handle Deletion of a Non-Existent Task
**As a** Priya Nair (team lead), **I want to** receive a clear message if I try to delete a task that has already been removed, **so that** I understand the state without seeing a confusing error page.

**Acceptance Criteria:**
- [ ] If the task ID does not exist at the time of the DELETE request, the server returns HTTP 404
- [ ] The user-facing message reads: "Task not found. It may have already been deleted."
- [ ] If the task ID is not a positive integer, the server returns HTTP 400 with: "Invalid task ID."

**Priority:** P0 | **Feature Ref:** F3

---

## Epic 4: Data Persistence (F4)

*Covers the infrastructure requirement that all task data is stored in a relational database, survives server restarts, and is written synchronously before any HTTP response is sent.*

---

### US-4.1: Trust That Tasks Persist Across Server Restarts
**As a** Marcus Webb (team member), **I want to** find all my tasks still present after the server is restarted or redeployed, **so that** I can trust TaskFlow as the single source of truth instead of re-entering tasks after downtime.

**Acceptance Criteria:**
- [ ] After a server restart, all tasks previously created are still returned on `GET /tasks`
- [ ] No task data is lost due to a server process restart
- [ ] The application reconnects to the database on startup without manual intervention
- [ ] Task data is never stored in application memory (module-level arrays, in-memory maps, etc.)

**Priority:** P0 | **Feature Ref:** F4

---

### US-4.2: Set Up the Database Schema from Scratch
**As a** developer on the team, **I want to** run a single migration command to create the database schema, **so that** any team member can set up a fresh environment without manually writing SQL.

**Acceptance Criteria:**
- [ ] A migration command is provided (e.g., `npx sequelize-cli db:migrate` or `python manage.py migrate`)
- [ ] Running the command creates the `tasks` table with columns: `id`, `title`, `description`, `status`, `due_date`, `created_at`, `updated_at`
- [ ] The `tasks` table has a CHECK constraint ensuring `status` is one of `'To Do'`, `'In Progress'`, `'Done'`
- [ ] The migration is version-controlled (not a one-off manual SQL command)
- [ ] The database can be switched between PostgreSQL (production) and SQLite (local dev) via `DATABASE_URL` environment variable only, with no code changes

**Priority:** P0 | **Feature Ref:** F4

---

### US-4.3: Receive Confirmation That My Changes Were Saved
**As a** Marcus Webb (team member), **I want to** be redirected to the task list only after my create, edit, or delete operation has been confirmed by the database, **so that** I never see a success redirect for an operation that silently failed.

**Acceptance Criteria:**
- [ ] The HTTP redirect (302) for create, edit, and delete is not sent until the database write/delete is fully confirmed (awaited)
- [ ] If a database write fails, the server returns HTTP 500 with: "Unable to save the task. Please try again."
- [ ] If a database delete fails, the server returns HTTP 500 with: "Unable to delete the task. Please try again."
- [ ] Error pages are served without data loss to the already-persisted records

**Priority:** P0 | **Feature Ref:** F4

---

## Epic 5: Status Filtering (F5)

*Covers the ability to narrow the task list to tasks of a specific status using URL-encoded filter controls that are bookmarkable and session-persistent.*

---

### US-5.1: Filter the Task List by Status
**As a** Priya Nair (team lead), **I want to** click a filter control to show only "In Progress" tasks, **so that** I can review active work each morning without scrolling through completed or backlog items.

**Acceptance Criteria:**
- [ ] Filter controls are rendered above the task list on the task list page at all times
- [ ] Four filter options are available: "All", "To Do", "In Progress", "Done"
- [ ] Clicking a filter option navigates to the corresponding URL (e.g., `GET /tasks?status=in_progress`)
- [ ] The task list returns only tasks matching the selected status
- [ ] The page returns HTTP 200 with the correct filtered results

**Priority:** P1 | **Feature Ref:** F5

---

### US-5.2: See the Active Filter Visually Indicated
**As a** Marcus Webb (team member), **I want to** see which filter is currently active, **so that** I always know whether I am looking at a filtered subset or the full task list.

**Acceptance Criteria:**
- [ ] The active filter option is visually distinguished from inactive options (e.g., bold, underline, filled button style)
- [ ] When no `?status=` parameter is present, the "All" option is shown as active
- [ ] The active indicator is driven by the URL parameter (not client-side state), so the page is refresh-safe
- [ ] Navigating directly to a bookmarked filtered URL shows the correct filter as active

**Priority:** P1 | **Feature Ref:** F5

---

### US-5.3: Bookmark or Share a Filtered Task List URL
**As a** Priya Nair (team lead), **I want to** bookmark the "In Progress" filtered URL and reopen it each morning, **so that** I have a persistent daily status view without re-applying the filter manually every time.

**Acceptance Criteria:**
- [ ] The URL for each filtered view encodes the active filter (e.g., `/tasks?status=in_progress`)
- [ ] Loading a bookmarked filtered URL returns the correctly filtered task list
- [ ] The filter state is derived entirely from the URL parameter (no server-side session required)
- [ ] The URL can be shared with another team member and they will see the same filtered view

**Priority:** P1 | **Feature Ref:** F5

---

### US-5.4: See an Empty State When No Tasks Match the Active Filter
**As a** Marcus Webb (team member), **I want to** see a descriptive empty state message when no tasks match my active filter, **so that** I know the filter is working and no tasks of that status currently exist.

**Acceptance Criteria:**
- [ ] When a filter is active and no tasks match, the message reads: "No tasks with status '[Status Label]'. Try a different filter or create a new task."
- [ ] `[Status Label]` reflects the human-readable label: "To Do", "In Progress", or "Done"
- [ ] Filter controls are still rendered in the empty state (user can switch to a different filter)
- [ ] The "New Task" button or link is still visible in the empty state
- [ ] An unknown or invalid `?status=` value silently falls back to the unfiltered "All" view with no error message shown

**Priority:** P1 | **Feature Ref:** F5

---

## Story Index

| Story ID | Title | Persona(s) | Priority | Feature Ref |
|----------|-------|------------|----------|-------------|
| US-0.1 | Create a New Task with a Title | PER-01 Marcus | P0 | F0 |
| US-0.2 | Create a Task with Optional Fields | PER-01 Marcus | P0 | F0 |
| US-0.3 | See Validation Errors on Invalid Task Creation | PER-01 Marcus | P0 | F0 |
| US-1.1 | View All Tasks at a Glance | PER-02 Priya | P0 | F1 |
| US-1.2 | Navigate from the Task List to a Task's Detail or Edit View | PER-01 Marcus | P0 | F1 |
| US-1.3 | See an Empty State When No Tasks Exist | PER-01 Marcus | P0 | F1 |
| US-2.1 | Open the Edit Form Pre-Populated with Current Task Values | PER-02 Priya | P0 | F2 |
| US-2.2 | Save Changes to a Task | PER-01 Marcus | P0 | F2 |
| US-2.3 | Update a Task's Status | PER-02 Priya | P0 | F2 |
| US-2.4 | See Validation Errors on Invalid Task Edit | PER-01 Marcus | P0 | F2 |
| US-3.1 | Delete a Task with a Confirmation Step | PER-01 Marcus | P0 | F3 |
| US-3.2 | Cancel a Deletion and Return to the Task List | PER-02 Priya | P0 | F3 |
| US-3.3 | Handle Deletion of a Non-Existent Task | PER-02 Priya | P0 | F3 |
| US-4.1 | Trust That Tasks Persist Across Server Restarts | PER-01 Marcus | P0 | F4 |
| US-4.2 | Set Up the Database Schema from Scratch | Developer | P0 | F4 |
| US-4.3 | Receive Confirmation That My Changes Were Saved | PER-01 Marcus | P0 | F4 |
| US-5.1 | Filter the Task List by Status | PER-02 Priya | P1 | F5 |
| US-5.2 | See the Active Filter Visually Indicated | PER-01 Marcus | P1 | F5 |
| US-5.3 | Bookmark or Share a Filtered Task List URL | PER-02 Priya | P1 | F5 |
| US-5.4 | See an Empty State When No Tasks Match the Active Filter | PER-01 Marcus | P1 | F5 |

---

## Priority Breakdown

| Priority | Count | Stories |
|----------|-------|---------|
| P0 — Critical | 16 | US-0.1 through US-4.3 |
| P1 — High | 4 | US-5.1 through US-5.4 |
| P2 — Medium | 0 | — |
| P3 — Low | 0 | — |
| **Total** | **20** | |

---

*Document generated: 2026-07-01 | Based on PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, PERSONAS-TaskFlow.md v1.0*
