# Story Map — TaskFlow
**Project Acronym:** TaskFlow
**Version:** 1.0
**Date:** 2026-07-01
**Status:** Draft
**Related Artifacts:** PRD-TaskFlow.md v1.0 · PERSONAS-TaskFlow.md v1.0 · JTBD-TaskFlow.md v1.0 · JOURNEYS-TaskFlow.md v1.0 · UserStories-TaskFlow.md v1.0

---

## Document Metadata

| Field | Value |
|-------|-------|
| Product | TaskFlow — Shared Task Tracker |
| Personas | PER-01 Marcus Webb (Team Member) · PER-02 Priya Nair (Team Lead) |
| Total User Stories | 20 (US-0.1 – US-5.4) |
| Releases Planned | R1: MVP Core (P0, 16 stories) · R2: UX Enhancement (P1, 4 stories) |
| Map Coverage | 20/20 stories mapped · 0 orphans |

---

## Overview

This Story Map organizes all 20 TaskFlow user stories into a two-dimensional grid:

- **X-axis (columns):** Journey stages drawn from JOURNEYS-TaskFlow.md — representing the sequential steps each persona takes to accomplish a job.
- **Y-axis (rows):** Activities and user stories nested within each stage, grouped by epic.
- **NaC column:** Natural Acceptance Criteria — testable criteria derived from the intersection of a JTBD outcome and a journey stage. NaC are *not invented*; each one traces directly back to a specific JTBD functional outcome applied to its journey context.
- **Release column:** Increment assignment (R1 or R2) based on PRD priority and journey completeness.

### How to Read the Map

1. **Follow a persona row** across the journey columns to understand their end-to-end experience.
2. **Read the NaC column** to understand what "done" looks like at each stage in JTBD terms.
3. **Filter by Release** to identify what ships in R1 (MVP) vs. R2 (UX Enhancement).
4. **Use the NaC Derivation Table** (Section 4) for full JTBD → journey stage → NaC traceability.

### NaC Concept

A Natural Acceptance Criterion bridges a JTBD job outcome to a testable story criterion:

```
JTBD Outcome (the "what matters") 
  + Journey Stage Context (the "when / where") 
  = NaC (the "how we verify it worked")
```

**Example:**
- JTBD-01.1 outcome: "Task captured in ≤60 seconds with zero friction"
- Journey stage: JRN-01.1 → Create
- NaC: "Given a user opens the form, when they submit a title, then the task appears in the shared list within 60 seconds and requires zero account setup"

---
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
---

## Release Planning

---

### R1: "MVP Core — Complete CRUD Journey" (P0 Stories)

**Theme:** Deliver a fully functional shared task tracker that enables both Marcus and Priya to complete their end-to-end journeys from first use to daily habit. Every P0 story ships in R1 so no journey is left half-complete.

**Release Rationale:** All 16 P0 stories are grouped into R1 because:
1. They form a complete journey dependency chain (create → view → edit → delete → persist).
2. No journey can be completed without the full CRUD foundation.
3. Filtering (F5, R2) enhances but does not block the core workflows — both personas can use the unfiltered list while R2 is in progress.

**Stories in R1:**

| SM-ID | US-ID | Story Title | Epic | Feature | Persona(s) | Journey Stage(s) |
|-------|-------|-------------|------|---------|-----------|-----------------|
| SM-0.1 | US-0.1 | Create a New Task with a Title | Epic 0 | F0 | PER-01 | JRN-01.1: Navigate → Create → Submit |
| SM-0.2 | US-0.2 | Create a Task with Optional Fields | Epic 0 | F0 | PER-01 | JRN-01.1: Create |
| SM-0.3 | US-0.3 | See Validation Errors on Invalid Task Creation | Epic 0 | F0 | PER-01 | JRN-01.1: Create (error path) |
| SM-1.1 | US-1.1 | View All Tasks at a Glance | Epic 1 | F1 | PER-01, PER-02 | JRN-01.2: Arrive/Orient · JRN-02.1: Scan/Assess |
| SM-1.2 | US-1.2 | Navigate from Task List to Detail/Edit View | Epic 1 | F1 | PER-01 | JRN-01.1: Submit · JRN-01.3: Open Edit |
| SM-1.3 | US-1.3 | See an Empty State When No Tasks Exist | Epic 1 | F1 | PER-01 | JRN-01.1: Return · JRN-01.2: Orient |
| SM-2.1 | US-2.1 | Open the Edit Form Pre-Populated | Epic 2 | F2 | PER-02 | JRN-02.1: Spot Risk · JRN-02.3: Edit Title |
| SM-2.2 | US-2.2 | Save Changes to a Task | Epic 2 | F2 | PER-01 | JRN-01.3: Update Status/Save |
| SM-2.3 | US-2.3 | Update a Task's Status | Epic 2 | F2 | PER-02 | JRN-02.3: Update Status |
| SM-2.4 | US-2.4 | See Validation Errors on Invalid Task Edit | Epic 2 | F2 | PER-01 | JRN-01.3: Open Edit |
| SM-3.1 | US-3.1 | Delete a Task with a Confirmation Step | Epic 3 | F3 | PER-01 | JRN-01.1: Return (cleanup) |
| SM-3.2 | US-3.2 | Cancel a Deletion and Return to Task List | Epic 3 | F3 | PER-02 | JRN-02.3: Delete Stale Task (cancel) |
| SM-3.3 | US-3.3 | Handle Deletion of a Non-Existent Task | Epic 3 | F3 | PER-02 | JRN-02.3: Delete Stale Task (race) |
| SM-4.1 | US-4.1 | Trust That Tasks Persist Across Server Restarts | Epic 4 | F4 | PER-01 | JRN-01.3: Verify |
| SM-4.2 | US-4.2 | Set Up the Database Schema from Scratch | Epic 4 | F4 | Developer | (setup) |
| SM-4.3 | US-4.3 | Receive Confirmation That My Changes Were Saved | Epic 4 | F4 | PER-01 | JRN-01.3: Save |

**Story Count:** 16 P0 stories

**Personas Served in R1:**
- ✅ PER-01 Marcus Webb — all 3 journeys (JRN-01.1, JRN-01.2, JRN-01.3) fully completable
- ✅ PER-02 Priya Nair — all 3 journeys (JRN-02.1, JRN-02.2, JRN-02.3) fully completable (with full list only; filter bookmarking deferred to R2)
- ✅ Developer — DB setup story included to unblock all other R1 stories

**JTBD Addressed in R1:**
- ✅ JTBD-01.1 — Capture new work immediately (US-0.1, US-0.2, US-0.3)
- ✅ JTBD-01.2 — Understand team work state without interrupting teammates (US-1.1, US-1.3 — full view; filter enhancement in R2)
- ✅ JTBD-01.3 — Progress a task and trust it will persist (US-2.2, US-2.4, US-4.1, US-4.3)
- ✅ JTBD-02.1 — Build accurate team status picture (US-1.1 — full list scan; filter shortcut in R2)
- ⚠️ JTBD-02.2 — Identify stuck/overdue work at a glance (partial: due date visible in US-1.1 but no In Progress filter in R1 — filter deferred to R2)
- ✅ JTBD-02.3 — Keep shared list clean and authoritative (US-2.1, US-2.3, US-3.1, US-3.2, US-3.3)

**R1 Journey Completeness:**

| Journey | Complete in R1? | Notes |
|---------|----------------|-------|
| JRN-01.1: Capture a new task | ✅ Full | Create → Submit → See in list → Persist |
| JRN-01.2: Morning orientation | ⚠️ Mostly | View all tasks (US-1.1) works; manual scan only — In Progress filter deferred to R2 |
| JRN-01.3: Progress a status update | ✅ Full | Find → Open Edit (pre-populated) → Save → Verify on list |
| JRN-02.1: Daily team status review | ⚠️ Mostly | Full list view works; bookmarked filter URL deferred to R2 |
| JRN-02.2: Identify stuck/overdue work | ⚠️ Mostly | Due date visible on rows; In Progress filter for triage deferred to R2 |
| JRN-02.3: Curate shared task list | ✅ Full | Edit title → Update status → Delete → Verify clean list |

---
---

### R2: "UX Enhancement — Filtering & Bookmarkability" (P1 Stories)

**Theme:** Add status filtering with bookmarkable URLs to complete Priya's daily status review workflow and reduce Marcus's morning orientation to a single bookmarked click. R2 stories all depend on R1's task list foundation (F1) being in place.

**Release Rationale:** F5 stories are P1 because:
1. The core CRUD journeys (R1) are fully functional without filtering — users can scroll the full list.
2. Filtering significantly improves Priya's daily habit (bookmarked "In Progress" URL) and removes noise for Marcus's morning scan.
3. All 4 F5 stories are tightly coupled (filter UI → active indicator → bookmarkable URL → empty state) and ship together as a coherent unit.

**Stories in R2:**

| SM-ID | US-ID | Story Title | Epic | Feature | Persona(s) | Journey Stage(s) |
|-------|-------|-------------|------|---------|-----------|-----------------|
| SM-5.1 | US-5.1 | Filter the Task List by Status | Epic 5 | F5 | PER-02 | JRN-02.1: Arrive · JRN-02.2: Focus |
| SM-5.2 | US-5.2 | See the Active Filter Visually Indicated | Epic 5 | F5 | PER-01 | JRN-01.2: Filter |
| SM-5.3 | US-5.3 | Bookmark or Share a Filtered Task List URL | Epic 5 | F5 | PER-02 | JRN-02.1: Arrive · JRN-02.2: Focus |
| SM-5.4 | US-5.4 | See an Empty State When No Tasks Match the Filter | Epic 5 | F5 | PER-01 | JRN-01.2: Filter |

**Story Count:** 4 P1 stories

**Dependencies:** All R2 stories depend on US-1.1 (Task List) and US-1.3 (Empty State) from R1 being complete and stable.

**Personas Served in R2:**
- ✅ PER-01 Marcus Webb — completes JRN-01.2 (Morning orientation with filter + visual indicator + empty state clarity)
- ✅ PER-02 Priya Nair — completes JRN-02.1 (Bookmarked "In Progress" URL as daily status view) and JRN-02.2 (Filtered overdue task identification)

**JTBD Addressed in R2 (completing what R1 started):**
- ✅ JTBD-01.2 — "Filter to In Progress in 1 click; bookmarkable URL reduces setup to 0 clicks" (US-5.2, US-5.4 complete this job)
- ✅ JTBD-02.1 — "Bookmarked filter URL opens pre-filtered In Progress view instantly" (US-5.1, US-5.3 complete this job)
- ✅ JTBD-02.2 — "Identify overdue In Progress tasks within 60 seconds via filtered view" (US-5.1, US-5.3 complete this job; due dates already visible from R1)

**R2 Journey Completeness:**

| Journey | Complete after R2? | What R2 Adds |
|---------|-------------------|--------------|
| JRN-01.2: Morning orientation | ✅ Full | Filter tab (US-5.1), active indicator (US-5.2), empty state on filter (US-5.4) |
| JRN-02.1: Daily team status review | ✅ Full | Filter to "In Progress" in 1 click (US-5.1); bookmarkable URL (US-5.3) |
| JRN-02.2: Identify stuck/overdue work | ✅ Full | "In Progress" filter isolates at-risk tasks for Priya's triage scan (US-5.1, US-5.3) |

**R2 Out-of-Scope Candidates (Deferred to v2+):**

The following items surfaced in JOURNEYS analysis as valued but are not in scope for either R1 or R2:

| Item | Journey Reference | Deferral Reason |
|------|-----------------|-----------------|
| Task assignee / ownership field | JRN-01.2, JRN-02.1, JRN-02.2, JRN-02.3 | Requires per-user identity (no auth in v1) |
| `updated_at` visible on task row | JRN-02.1: Assess, JRN-02.2: Scan Due Dates | Low-cost improvement but not a P1 blocker |
| Save-confirmation toast | JRN-01.3: Save, JRN-02.3: Verify | Nice-to-have UX polish; trust built implicitly via redirect |
| Comments / activity log | JRN-02.2: Identify Overdue → Validate | Deferred complexity; CRUD-only v1 |
| Printable / copy-pasteable status summary | JRN-02.1: Prepare | v2 export feature |

---
---

## Coverage Analysis

---

### Persona Coverage by Release

| Persona | R1 Coverage | R2 Coverage | Fully Served After |
|---------|------------|------------|-------------------|
| PER-01 Marcus Webb | JRN-01.1 ✅ Full · JRN-01.2 ⚠️ Partial (no filter) · JRN-01.3 ✅ Full | JRN-01.2 ✅ Complete | R2 |
| PER-02 Priya Nair | JRN-02.1 ⚠️ Partial (no bookmark filter) · JRN-02.2 ⚠️ Partial (no filter) · JRN-02.3 ✅ Full | JRN-02.1 ✅ · JRN-02.2 ✅ Complete | R2 |
| Developer | US-4.2 ✅ (DB setup) | — | R1 |

**R1 alone:** Both personas can use TaskFlow for daily work. Filtering is absent but not blocking — full list scrolling serves both journeys.
**R2 complete:** All journeys fully served. Priya's bookmark habit and Marcus's focused morning scan are both enabled.

---

### JTBD Coverage by Release

| JTBD-ID | Job (abbreviated) | R1 Coverage | R2 Coverage | First Complete |
|---------|------------------|------------|------------|---------------|
| JTBD-01.1 | Capture new work immediately | ✅ US-0.1, US-0.2, US-0.3 | — | R1 |
| JTBD-01.2 | Understand team work state without interrupting | ⚠️ US-1.1 (list view only) | ✅ US-5.2, US-5.4 | R2 |
| JTBD-01.3 | Progress a task and trust it will persist | ✅ US-2.2, US-2.4, US-4.1, US-4.3 | — | R1 |
| JTBD-02.1 | Build accurate team status picture | ⚠️ US-1.1 (full list only) | ✅ US-5.1, US-5.3 | R2 |
| JTBD-02.2 | Identify stuck/overdue work at a glance | ⚠️ US-1.1 (due date visible, no filter) | ✅ US-5.1, US-5.3, US-5.4 | R2 |
| JTBD-02.3 | Keep shared list clean and authoritative | ✅ US-2.1, US-2.3, US-3.1, US-3.2, US-3.3 | — | R1 |

**4 of 6 JTBD fully addressed in R1.** JTBD-01.2 and JTBD-02.1/02.2 are partially addressed in R1 and completed in R2.

---

### Journey Stage Coverage

| Journey | Stage | Covered By | Release |
|---------|-------|------------|---------|
| JRN-01.1 | Triggered | (external — Slack; no story needed) | — |
| JRN-01.1 | Navigate | US-0.1 (New Task button visible) | R1 |
| JRN-01.1 | Create | US-0.1, US-0.2, US-0.3 | R1 |
| JRN-01.1 | Submit | US-0.1, US-1.2, US-4.3 | R1 |
| JRN-01.1 | Return | US-4.1 (persistence trust), US-3.1 (cleanup) | R1 |
| JRN-01.2 | Arrive | US-1.1 | R1 |
| JRN-01.2 | Orient | US-1.1, US-1.3 | R1 |
| JRN-01.2 | Filter | US-5.2, US-5.4 | R2 |
| JRN-01.2 | Scan | US-1.1 | R1 |
| JRN-01.2 | Complete | (mental task; no story — covered by R1 list) | — |
| JRN-01.3 | Identify | US-1.1 (task list scan) | R1 |
| JRN-01.3 | Open Edit | US-2.1, US-2.4, US-1.2 | R1 |
| JRN-01.3 | Update Status | US-2.2, US-2.3 | R1 |
| JRN-01.3 | Save | US-2.2, US-4.3 | R1 |
| JRN-01.3 | Verify | US-1.1, US-4.1 | R1 |
| JRN-02.1 | Arrive | US-5.3 (bookmark) | R2 |
| JRN-02.1 | Scan | US-1.1 | R1 |
| JRN-02.1 | Spot Risk | US-1.1 (due date visible on row) | R1 |
| JRN-02.1 | Assess | US-1.1 | R1 |
| JRN-02.1 | Prepare | (mental task; no story needed) | — |
| JRN-02.1 | Act | (external — Slack; no story needed) | — |
| JRN-02.2 | Focus | US-5.1, US-5.3 | R2 |
| JRN-02.2 | Scan Due Dates | US-1.1 (due date column), US-5.4 (empty state) | R1/R2 |
| JRN-02.2 | Identify Overdue | US-5.1 (filter active), US-1.1 | R1/R2 |
| JRN-02.2 | Validate | US-2.1 (open edit to read description) | R1 |
| JRN-02.2 | Follow Up | US-1.1 (task list reference) | R1 |
| JRN-02.3 | Survey | US-1.1 (all tasks view) | R1 |
| JRN-02.3 | Edit Title | US-2.1, US-2.2 | R1 |
| JRN-02.3 | Update Status | US-2.3 | R1 |
| JRN-02.3 | Delete Stale Task | US-3.1, US-3.2, US-3.3 | R1 |
| JRN-02.3 | Verify Clean List | US-1.1, US-4.1 | R1 |

**4 stages marked as external or mental tasks (no story required):**
- JRN-01.1: Triggered (Marcus reads Slack — external)
- JRN-01.2: Complete (Marcus opens IDE — mental/external)
- JRN-02.1: Prepare (Priya composes summary — mental)
- JRN-02.1: Act (Priya messages in Slack — external)

---

### Gap Analysis

#### Journey Stages Without Story Coverage
**None.** All 27 product-facing journey stages have at least one mapped story. The 4 external/mental stages require no stories by design.

#### JTBD Outcomes Without Any NaC
**None.** All 6 JTBD outcomes have at least one derived NaC (22 NaC total across 20 stories).

#### Orphan Stories (Not Mapped to Any Journey Stage)
**None.** All 20 user stories (US-0.1 – US-5.4) are mapped to at least one journey stage. US-4.2 (Developer setup) maps to the database foundation that underpins all journeys rather than a specific persona stage — this is expected and noted, not a gap.

#### Known v1 Gaps Accepted by Design

| Gap | Journey Reference | v1 Mitigation | v2 Candidate |
|-----|-----------------|--------------|-------------|
| No task assignee/ownership | JRN-01.2, JRN-02.1, JRN-02.2 | Task title naming convention | Task assignment field |
| No `updated_at` visible on task row | JRN-02.1, JRN-02.2 | Click into task to see edit form | `updated_at` column on task list row |
| No save-confirmation toast | JRN-01.3, JRN-02.3 | Redirect to list with updated values implicit | Visual save toast or success banner |
| No activity log / comment thread | JRN-02.2: Validate | Description field as workaround | Comments / activity history |
| No real-time updates | JRN-01.2: Complete | Team update discipline; page refresh | WebSocket / polling |

---
