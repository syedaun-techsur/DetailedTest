# Jobs-to-be-Done Document — TaskFlow
**Project Acronym:** TaskFlow
**Version:** 1.0
**Date:** 2026-07-01
**Status:** Draft
**Related Personas:** PERSONAS-TaskFlow.md v1.0
**Related PRD:** PRD-TaskFlow.md v1.0
**Derived From:** Persona goals, pain points, top tasks; PRD features F0–F5 and success metrics

---

## JTBD Summary Table

| JTBD-ID | Persona | Job Statement (abbreviated) | Priority |
|---------|---------|----------------------------|----------|
| JTBD-01.1 | PER-01 Marcus (Team Member) | Capture new work the moment it surfaces, without friction or setup | P0 |
| JTBD-01.2 | PER-01 Marcus (Team Member) | Understand current team work state without disrupting teammates | P0 |
| JTBD-01.3 | PER-01 Marcus (Team Member) | Move a task forward in status and trust the record will persist | P0 |
| JTBD-02.1 | PER-02 Priya (Team Lead) | Build an accurate team status picture without chasing people in chat | P0 |
| JTBD-02.2 | PER-02 Priya (Team Lead) | Identify and triage stuck or overdue work at a glance each morning | P1 |
| JTBD-02.3 | PER-02 Priya (Team Lead) | Keep the shared task list clean and authoritative by removing clutter | P0 |

---

## PER-01: Marcus Webb — Team Member

---

### JTBD-01.1: Capture New Work Immediately

**Job Statement:**
When a new to-do surfaces during standup, a Slack message, or while reviewing code, I want to log it in the shared task list immediately with minimal steps, so I can return to my work knowing the item is captured and visible to teammates without requiring me to create an account or navigate an onboarding flow.

**Current Alternatives:**
- Pins a message in a Slack thread that becomes buried within hours
- Adds a line to a local text file that goes out of sync with team reality by the next morning
- Relies on memory or verbal commitment, which causes tasks to be missed or duplicated

**Hiring Criteria:**
- Task creation requires no login, account setup, or guided tour
- Only a title is required — description, status, and due date are optional
- The new task appears immediately in the shared list visible to all teammates
- The entire creation flow completes in 3 clicks or fewer from the task list

**Success Measure:** A new team member creates their first task within 60 seconds of navigating to the application with no instructions.

**Related Features:** F0, F4
**Priority:** P0

---

### JTBD-01.2: Understand Team Work State Without Interrupting Teammates

**Job Statement:**
When I sit down to start my day or pick up a new piece of work, I want to scan the current state of all team tasks at a glance, so I can identify what is already in progress or done and avoid duplicating effort or stepping on a teammate's work — without having to ask anyone in chat.

**Current Alternatives:**
- Sends a "are you still working on X?" message in Slack, which is disruptive and slow
- Cross-references a shared Google Doc or spreadsheet that is frequently stale
- Relies on the previous day's verbal standup, which may no longer reflect current reality

**Hiring Criteria:**
- All tasks are visible in a single scrollable list with status shown inline
- Status labels (To Do / In Progress / Done) are distinguishable at a glance without opening individual tasks
- List can be filtered to "In Progress" to reduce noise when checking for active work
- Page loads within 2 seconds so the check-in feels instant, not costly

**Success Measure:** Marcus can determine the current status of any active task within 10 seconds of opening the application.

**Related Features:** F1, F5
**Priority:** P0

---

### JTBD-01.3: Progress a Task and Trust It Will Persist

**Job Statement:**
When work moves forward — a task is picked up, completed, or cancelled — I want to update its status quickly and be confident the change is saved to a real database, so I can stop maintaining a separate personal to-do list and rely on TaskFlow as the one place that reflects actual work state.

**Current Alternatives:**
- Updates a local text file or Slack pin, which is not visible to teammates
- Uses an in-memory tool that loses data on server restart, forcing re-entry and eroding trust
- Leaves tasks in an outdated state because updating a spreadsheet feels like overhead

**Hiring Criteria:**
- Status update (e.g., "To Do" → "In Progress" → "Done") completable in 2 clicks from the task list
- Any field on a task is editable — title, description, status, due date — from a pre-populated form
- Changes are written to the database immediately and survive a server restart
- No data loss over 30 days of normal use

**Success Measure:** Zero task data loss due to server restart or application error over the first 30 days of use.

**Related Features:** F2, F4
**Priority:** P0

---

## PER-02: Priya Nair — Team Lead / Coordinator

---

### JTBD-02.1: Build an Accurate Team Status Picture Without Chasing People

**Job Statement:**
When I open the task list at the start of my workday or before a Friday status update to my department head, I want to see an up-to-date view of what every team member is working on, so I can give an accurate verbal status summary in under 2 minutes without having to ask anyone for an update in chat.

**Current Alternatives:**
- Manually chases team members in Slack or chat for current status before each standup
- Reads a shared Google Sheet that contributors skip updating under deadline pressure
- Relies on a patchwork of verbal updates and buried Slack threads that cannot be easily shared with stakeholders

**Hiring Criteria:**
- Task list shows all team tasks with title, status, and due date visible without clicking into each task
- Status filter ("In Progress") isolates active work in a single click
- Filtered URL is bookmarkable so Priya can return to her daily status view without reconfiguring filters
- List reflects the current state — no stale items from contributors who skipped a spreadsheet update

**Success Measure:** Priya can produce a verbal team status update in under 2 minutes by scanning the filtered task list.

**Related Features:** F1, F5, F4
**Priority:** P0

---

### JTBD-02.2: Identify Stuck or Overdue Work at a Glance

**Job Statement:**
When I review the team's active tasks each morning, I want to quickly spot tasks that have been sitting in "In Progress" with a past due date or no recent update, so I can proactively follow up with the right teammate rather than discovering a blocker during a stakeholder check-in.

**Current Alternatives:**
- Scans a spreadsheet looking for rows that haven't changed since the last manual update cycle
- Relies on verbal standup reports that may not surface tasks quietly abandoned or stuck
- Has no systematic way to distinguish "actively in progress" from "quietly abandoned" in current tools

**Hiring Criteria:**
- Due date is visible on each task row in the list without opening the task
- Status filter isolates "In Progress" tasks so overdue items with that status stand out
- Filter state is preserved in the URL so Priya can link the filtered view to a standup agenda
- Empty state is shown clearly when a filter returns no results (not a confusing blank page)

**Success Measure:** Priya can identify all overdue or stuck "In Progress" tasks within 60 seconds of opening the filtered task list.

**Related Features:** F1, F5
**Priority:** P1

---

### JTBD-02.3: Keep the Shared List Clean and Authoritative

**Job Statement:**
When a task is cancelled, completed by verbal agreement, or entered incorrectly by a team member, I want to update or remove it without needing to contact the original creator or obtain any special permissions, so I can maintain the shared list as the team's single authoritative source of truth rather than a secondary copy of a spreadsheet.

**Current Alternatives:**
- Leaves stale tasks in the spreadsheet because correcting them requires messaging the original contributor
- Maintains a separate "archived" tab in the spreadsheet that adds complexity and reduces clarity
- Tolerates an increasingly noisy list that loses credibility as a source of truth over time

**Hiring Criteria:**
- Any task can be edited by any team member (no per-user ownership or permission check in v1)
- Edit form pre-populates all existing values so corrections require only changing the relevant field
- Delete action is available from the task list with a confirmation step to prevent accidental removal
- Deleted tasks disappear immediately from the list for all viewers — no ghost rows or caching artifacts

**Success Measure:** All team members (100%) use TaskFlow as the primary task tracking tool within 2 weeks of launch, eliminating the need for a parallel spreadsheet.

**Related Features:** F2, F3, F4
**Priority:** P0

---

## Outcome-to-Feature Traceability

| JTBD-ID | Related Feature(s) | Expected Outcome |
|---------|--------------------|-----------------|
| JTBD-01.1 | F0 Task Creation, F4 Data Persistence | New task captured in under 60 seconds with zero friction; immediately visible to all teammates |
| JTBD-01.2 | F1 Task List View, F5 Status Filtering | Team work state readable in under 10 seconds; duplicated effort eliminated |
| JTBD-01.3 | F2 Task Edit, F4 Data Persistence | Status updated in 2 clicks; zero data loss over 30 days |
| JTBD-02.1 | F1 Task List View, F5 Status Filtering, F4 Data Persistence | Verbal status update producible in under 2 minutes without chasing teammates |
| JTBD-02.2 | F1 Task List View, F5 Status Filtering | Overdue/stuck tasks identified within 60 seconds of opening filtered view |
| JTBD-02.3 | F2 Task Edit, F3 Task Deletion, F4 Data Persistence | Shared list maintained as single source of truth; parallel spreadsheet eliminated within 2 weeks |

**Feature Coverage Verification:**

| Feature ID | Covered by JTBD |
|------------|----------------|
| F0 | JTBD-01.1 |
| F1 | JTBD-01.2, JTBD-02.1, JTBD-02.2 |
| F2 | JTBD-01.3, JTBD-02.3 |
| F3 | JTBD-02.3 |
| F4 | JTBD-01.1, JTBD-01.3, JTBD-02.1, JTBD-02.3 |
| F5 | JTBD-01.2, JTBD-02.1, JTBD-02.2 |

All 6 PRD features (F0–F5) are covered by at least one JTBD.

---

## NaC Preview

Candidate Natural Acceptance Criteria derived from job success measures. These will be refined into full NaC statements downstream (STORY-MAP).

| JTBD-ID | Outcome | Candidate Natural Acceptance Criteria |
|---------|---------|---------------------------------------|
| JTBD-01.1 | First task created in ≤60 seconds with no instructions | **Given** a new user opens the application for the first time, **When** they fill in a task title and submit, **Then** the task appears in the shared list within 60 seconds and requires zero account setup |
| JTBD-01.2 | Any active task's status readable within 10 seconds | **Given** the task list is open, **When** a user scans or filters the list, **Then** the status of any active task is visible without clicking into the task detail, and the page loads in under 2 seconds |
| JTBD-01.3 | Zero data loss over 30 days; status update in ≤2 clicks | **Given** a task exists, **When** the server is restarted, **Then** all tasks remain intact; **And Given** a user changes a task status, **Then** the update is saved in ≤2 clicks from the task list |
| JTBD-02.1 | Verbal status update producible in ≤2 minutes | **Given** Priya opens the filtered "In Progress" view, **When** she scans the list, **Then** she can identify owner-agnostic task titles, statuses, and due dates without opening any individual task |
| JTBD-02.2 | Overdue/stuck tasks identified in ≤60 seconds | **Given** the "In Progress" filter is active, **When** tasks with past due dates are present, **Then** due dates are visible inline on the task row so overdue items are identifiable without additional clicks |
| JTBD-02.3 | 100% team adoption within 2 weeks; parallel spreadsheet eliminated | **Given** a task was entered with incorrect details, **When** any team member edits and saves the task, **Then** the corrected values are immediately visible to all viewers with no permission prompt; **And** delete with confirmation removes the task permanently for all viewers |

---

## Validation Checklist

- [x] Every persona (PER-01, PER-02) has at least 2 jobs (PER-01: 3 jobs; PER-02: 3 jobs)
- [x] Every job uses "When / I want / So I can" format
- [x] Hiring criteria are specific and testable (click counts, load times, concrete behaviors)
- [x] Success measures are quantifiable (seconds, percentages, day counts)
- [x] All PRD features F0–F5 appear in Outcome-to-Feature Traceability
- [x] No two jobs are redundant across personas (Marcus focuses on individual capture/update; Priya focuses on team visibility/curation)
- [x] NaC Preview table is complete (6 entries covering all 6 jobs)

---

*Document generated: 2026-07-01 | Derived from PERSONAS-TaskFlow.md v1.0 and PRD-TaskFlow.md v1.0*
