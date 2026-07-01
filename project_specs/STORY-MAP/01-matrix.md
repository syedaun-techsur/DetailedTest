---

## Story Map Matrix

> **Reading key:** Each row is a user story placed at its primary journey stage. The NaC is derived from the JTBD outcome most relevant to that stage. SM IDs use `SM-{Epic}.{NN}` format.

---

### PER-01: Marcus Webb — Team Member

Marcus's journeys: JRN-01.1 (Capture new task), JRN-01.2 (Morning orientation), JRN-01.3 (Progress & trust status update)

| SM-ID | Journey Stage | Activity | Epic | Story | NaC (JTBD Source) | Release |
|-------|--------------|----------|------|-------|-------------------|---------|
| SM-0.1 | JRN-01.1: Navigate | Open TaskFlow in new tab, reach task list | Epic 0 (F0) | **US-0.1:** Create a New Task with a Title | JTBD-01.1 → "Task captured in ≤60 seconds": A "New Task" button is visible on the task list page so Marcus reaches the creation form in 1 click | R1 |
| SM-0.2 | JRN-01.1: Create | Enter title; optionally add description, status, due date | Epic 0 (F0) | **US-0.2:** Create a Task with Optional Fields | JTBD-01.1 → "Only title required, optional fields do not block": Submitting with title only succeeds; description and due date may be omitted with no error | R1 |
| SM-0.3 | JRN-01.1: Create | Submit form with incomplete or invalid input | Epic 0 (F0) | **US-0.3:** See Validation Errors on Invalid Task Creation | JTBD-01.1 → "Creation flow completes in 3 clicks or fewer": Inline errors appear without losing entered data so Marcus corrects and resubmits in one cycle | R1 |
| SM-1.2 | JRN-01.1: Submit | Click task row to navigate to detail or edit | Epic 1 (F1) | **US-1.2:** Navigate from Task List to Detail/Edit View | JTBD-01.1 → "New task immediately visible": Each task row is a clickable link so Marcus can confirm or open a task in 1 click from the list | R1 |
| SM-1.3 | JRN-01.1: Return / JRN-01.2: Orient | Open app for first time or see empty list | Epic 1 (F1) | **US-1.3:** See an Empty State When No Tasks Exist | JTBD-01.2 → "Page load feels instant, not confusing": A clear "No tasks yet" message is shown so Marcus knows the list is empty, not broken | R1 |
| SM-1.1 | JRN-01.2: Arrive / Orient | Open task list to see all team work at a glance | Epic 1 (F1) | **US-1.1:** View All Tasks at a Glance | JTBD-01.2 → "Status of any active task readable within 10 seconds": Page loads within 2 seconds; each row shows title, status badge, due date, and description preview without opening the task | R1 |
| SM-2.2 | JRN-01.3: Update Status / Save | Edit a task's fields and save | Epic 2 (F2) | **US-2.2:** Save Changes to a Task | JTBD-01.3 → "Status update in ≤2 clicks; changes survive restart": Submitting valid edits redirects to task list showing updated values; `updated_at` is stamped immediately | R1 |
| SM-2.4 | JRN-01.3: Open Edit | Submit edit form with invalid data | Epic 2 (F2) | **US-2.4:** See Validation Errors on Invalid Task Edit | JTBD-01.3 → "Edit form pre-populated so only changed field needs updating": Re-renders with submitted values intact so Marcus fixes one field without re-entering others | R1 |
| SM-3.1 | JRN-01.1: Return (cleanup) | Delete an erroneous or cancelled task | Epic 3 (F3) | **US-3.1:** Delete a Task with a Confirmation Step | JTBD-02.3 → "Delete with confirmation removes task permanently for all viewers": Confirmation dialog names the specific task; on confirm, task is permanently removed and list refreshes immediately | R1 |
| SM-4.1 | JRN-01.3: Verify | Trust that tasks survive server restart | Epic 4 (F4) | **US-4.1:** Trust That Tasks Persist Across Server Restarts | JTBD-01.3 → "Zero data loss over 30 days": After server restart all previously created tasks are still returned on `GET /tasks`; no in-memory storage is used | R1 |
| SM-4.3 | JRN-01.3: Save | Receive confirmation that a change was saved | Epic 4 (F4) | **US-4.3:** Receive Confirmation That My Changes Were Saved | JTBD-01.3 → "Changes written to DB immediately, survive restart": HTTP 302 redirect is not sent until DB write is fully confirmed; DB failure returns HTTP 500 | R1 |
| SM-5.2 | JRN-01.2: Filter | See which filter is currently active | Epic 5 (F5) | **US-5.2:** See the Active Filter Visually Indicated | JTBD-01.2 → "Filter to In Progress in 1 click": Active filter is visually distinguished and driven by URL parameter so Marcus always knows if he is looking at a filtered subset | R2 |
| SM-5.4 | JRN-01.2: Filter | See empty state when filter returns nothing | Epic 5 (F5) | **US-5.4:** See an Empty State When No Tasks Match the Active Filter | JTBD-02.2 → "Empty state shown clearly when filter returns no results": Descriptive message and filter controls remain visible so Marcus knows the filter is working, not broken | R2 |

---

### PER-02: Priya Nair — Team Lead / Coordinator

Priya's journeys: JRN-02.1 (Daily team status review), JRN-02.2 (Identify stuck/overdue work), JRN-02.3 (Curate shared task list)

| SM-ID | Journey Stage | Activity | Epic | Story | NaC (JTBD Source) | Release |
|-------|--------------|----------|------|-------|-------------------|---------|
| SM-1.1* | JRN-02.1: Scan / Assess | Scan all tasks with inline title, status, due date | Epic 1 (F1) | **US-1.1:** View All Tasks at a Glance | JTBD-02.1 → "Verbal status update producible in ≤2 minutes": Title, status, and due date all visible on each row without opening any task; page loads within 2 seconds | R1 |
| SM-2.1 | JRN-02.1: Spot Risk / JRN-02.3: Edit Title | Open edit form for a task to update or correct it | Epic 2 (F2) | **US-2.1:** Open the Edit Form Pre-Populated with Current Task Values | JTBD-02.3 → "Any field editable by any team member; no permission prompt": Edit form at `GET /tasks/:id/edit` pre-populates all current values so Priya changes only the field that needs correcting | R1 |
| SM-2.3 | JRN-02.3: Update Status | Change a task's status on behalf of a team member | Epic 2 (F2) | **US-2.3:** Update a Task's Status | JTBD-02.3 → "Status corrected on behalf of team member in ≤2 clicks": Selecting a new status and saving updates the badge immediately on the task list for all viewers | R1 |
| SM-3.2 | JRN-02.3: Delete Stale Task | Cancel a deletion after clicking Delete by mistake | Epic 3 (F3) | **US-3.2:** Cancel a Deletion and Return to the Task List | JTBD-02.3 → "Confirmation dialog prevents accidental removal": Cancel option in the confirmation step returns Priya to `GET /tasks` with no changes; task remains intact | R1 |
| SM-3.3 | JRN-02.3: Delete Stale Task | Encounter a task that was already deleted | Epic 3 (F3) | **US-3.3:** Handle Deletion of a Non-Existent Task | JTBD-02.3 → "Deleted tasks disappear immediately for all viewers": HTTP 404 with clear message so Priya understands the state without a confusing error page | R1 |
| SM-4.2 | (Developer story — setup) | Run migration to create DB schema from scratch | Epic 4 (F4) | **US-4.2:** Set Up the Database Schema from Scratch | JTBD-01.3 / JTBD-02.3 → "Changes written to real DB, survive restart": Single migration command creates `tasks` table with all required columns and status CHECK constraint | R1 |
| SM-5.1 | JRN-02.1: Arrive / JRN-02.2: Focus | Apply "In Progress" filter to isolate active work | Epic 5 (F5) | **US-5.1:** Filter the Task List by Status | JTBD-02.1 → "In Progress filter isolates active work in a single click": Clicking "In Progress" filter navigates to `/tasks?status=in_progress` returning only matching tasks | R2 |
| SM-5.3 | JRN-02.1: Arrive / JRN-02.2: Focus | Bookmark filtered URL for daily use | Epic 5 (F5) | **US-5.3:** Bookmark or Share a Filtered Task List URL | JTBD-02.1 → "Bookmarked filter URL opens pre-filtered view instantly": `/tasks?status=in_progress` is bookmarkable; filter state is derived from URL parameter with no server session | R2 |

> *SM-1.1 appears in both persona sections because US-1.1 (View All Tasks) is primary for both PER-01 and PER-02 and maps to multiple journeys.

---

### Complete Story Placement Index

| SM-ID | US-ID | Persona(s) | Journey(s) | Stage(s) | Release |
|-------|-------|-----------|------------|----------|---------|
| SM-0.1 | US-0.1 | PER-01 | JRN-01.1 | Navigate → Create | R1 |
| SM-0.2 | US-0.2 | PER-01 | JRN-01.1 | Create | R1 |
| SM-0.3 | US-0.3 | PER-01 | JRN-01.1 | Create | R1 |
| SM-1.1 | US-1.1 | PER-01, PER-02 | JRN-01.2, JRN-02.1 | Arrive, Orient, Scan, Assess | R1 |
| SM-1.2 | US-1.2 | PER-01 | JRN-01.1, JRN-01.3 | Submit, Open Edit | R1 |
| SM-1.3 | US-1.3 | PER-01 | JRN-01.1, JRN-01.2 | Return, Orient | R1 |
| SM-2.1 | US-2.1 | PER-02 | JRN-02.1, JRN-02.3 | Spot Risk, Edit Title | R1 |
| SM-2.2 | US-2.2 | PER-01 | JRN-01.3 | Update Status, Save | R1 |
| SM-2.3 | US-2.3 | PER-02 | JRN-02.3 | Update Status | R1 |
| SM-2.4 | US-2.4 | PER-01 | JRN-01.3 | Open Edit | R1 |
| SM-3.1 | US-3.1 | PER-01 | JRN-01.1 | Return (cleanup) | R1 |
| SM-3.2 | US-3.2 | PER-02 | JRN-02.3 | Delete Stale Task | R1 |
| SM-3.3 | US-3.3 | PER-02 | JRN-02.3 | Delete Stale Task | R1 |
| SM-4.1 | US-4.1 | PER-01 | JRN-01.3 | Verify | R1 |
| SM-4.2 | US-4.2 | Developer | (setup) | — | R1 |
| SM-4.3 | US-4.3 | PER-01 | JRN-01.3 | Save | R1 |
| SM-5.1 | US-5.1 | PER-02 | JRN-02.1, JRN-02.2 | Arrive, Focus | R2 |
| SM-5.2 | US-5.2 | PER-01 | JRN-01.2 | Filter | R2 |
| SM-5.3 | US-5.3 | PER-02 | JRN-02.1, JRN-02.2 | Arrive, Focus | R2 |
| SM-5.4 | US-5.4 | PER-01 | JRN-01.2 | Filter | R2 |

**Total mapped: 20/20 stories. Orphans: 0.**

---
