# Personas Document — TaskFlow
**Project Acronym:** TaskFlow
**Version:** 1.0
**Date:** 2026-07-01
**Status:** Draft
**Related PRD:** PRD-TaskFlow.md v1.0
**Derived From:** PRD Section 2 (Problem Statement), PRD Section 5 (Feature Requirements), PROJECT.md

---

## Persona Summary Table

| PER-ID | Name | Role | Primary Goal |
|--------|------|------|--------------|
| PER-01 | Marcus Webb | Team Member (Individual Contributor) | Quickly create and update tasks without friction so work stays visible to the team |
| PER-02 | Priya Nair | Team Lead / Coordinator | Get an accurate snapshot of team workload at a glance and keep the task list clean and current |

---

## PER-01: Marcus Webb — Team Member

**Role & Context:**
Marcus is a developer on a 6-person internal tools team. He contributes across several concurrent workstreams and needs a lightweight way to log what he is working on and mark things done. He has no dedicated project management background and considers task tracking a means to an end, not a discipline in itself. Throughout the day Marcus switches between his IDE, a browser tab with the team task list, and a chat tool. He currently captures to-dos in a pinned Slack message or a local text file — both of which drift out of sync with reality within 24 hours. He is comfortable with web apps and expects tools to behave like standard forms with no learning curve.

**Goals:**
- Log new work items immediately when they surface, without navigating an onboarding flow or creating an account (F0, PRD §3 "adopt in minutes with no onboarding")
- See at a glance what is in progress vs. done so he doesn't duplicate effort with teammates (F1, F5)
- Update a task's status in two clicks when work moves forward (F2)
- Remove completed or cancelled tasks that are no longer relevant (F3)
- Trust that what he entered yesterday is still there today — no re-entry after server downtime (F4)

**Pain Points:**
- Shared Slack threads and Google Docs go stale fast; no single source of truth for team task status (PRD §2 "Status drift")
- Commercial tools like Jira require account setup and onboarding that feels disproportionate to a 6-person internal project (PRD §2 "Over-engineered alternatives")
- Asking teammates "are you still working on X?" to get a status update is disruptive and slow (PRD §2 "No shared visibility")
- Lightweight alternatives that lose data on restart erode trust in the tool entirely (PRD §2, PRD §6 Reliability)

**Technical Expertise:** Intermediate — comfortable with any web UI, navigates forms and filters without help, does not need tooltips or guided tours

**Top Tasks:**
1. Create a new task with a title and optional due date (daily, critical) — F0
2. Change a task's status from "To Do" to "In Progress" or "Done" (daily, critical) — F2
3. Scan the full task list to see what teammates are working on (daily, high) — F1
4. Filter the list to "In Progress" tasks to focus attention (several times/week, high) — F5
5. Delete a task that was cancelled or entered in error (occasional, medium) — F3

**Success Criteria:**
- Creates a first task in under 60 seconds with no instructions (PRD §7 "Time-to-First-Task")
- Can determine the current status of any active task within 10 seconds of opening the app (PRD §7 "Task Completion Visibility")
- Zero task data lost due to server restart over the first 30 days (PRD §7 "Data Reliability")
- All core CRUD operations completable in 3 clicks or fewer from the task list (PRD §6 Usability)

---

## PER-02: Priya Nair — Team Lead / Coordinator

**Role & Context:**
Priya leads a small internal team of 8 engineers and designers. She is not a dedicated project manager — she carries her own individual contributor work alongside coordination responsibilities. Her coordination tasks run on a lightweight daily cadence: she scans the shared task list each morning to identify blockers, checks in with team members on stale items, and gives a brief status update to her department head each Friday. She currently maintains a shared Google Sheet to track team workload, but it is updated inconsistently — contributors forget to update it, leaving Priya to chase status in chat. She is comfortable with web tools and spreadsheets, and she values clarity and speed over feature richness in any coordination tool she adopts.

**Goals:**
- Get an accurate, up-to-date snapshot of team workload without having to chase team members in chat (F1, F5, PRD §2 "No shared visibility")
- Quickly identify tasks that are overdue or stuck in "In Progress" for too long (F1, F5)
- Correct or update task details when a team member has entered them incorrectly or incompletely (F2)
- Remove stale or obsolete tasks that clutter the list and reduce signal-to-noise (F3)
- Rely on the task list as the single authoritative source of truth — not a secondary copy of a spreadsheet (F4, PRD §3 "eliminate the gap between agreed and visible")

**Pain Points:**
- Shared spreadsheets require manual updates that contributors skip under deadline pressure, leaving Priya with a stale picture of team status (PRD §2 "Status drift")
- No visibility into what is blocked vs. actively in progress vs. quietly abandoned (PRD §2 "No shared visibility")
- Commercial PM tools (Jira, Asana) require admin setup, user provisioning, and ongoing maintenance that she cannot justify for a team of 8 on an internal workstream (PRD §2 "Over-engineered alternatives")
- Status updates from chat threads are buried in history and cannot be easily shared with stakeholders (PRD §2)

**Technical Expertise:** Intermediate — very comfortable with web apps and spreadsheets, uses filters and bookmarked URLs as workflow habits, does not require power-user features

**Top Tasks:**
1. Open the task list and filter by "In Progress" to review active work each morning (daily, critical) — F1, F5
2. Edit a task's title or description to correct inaccurate information entered by a team member (several times/week, high) — F2
3. Change a task's status to "Done" on behalf of a team member after verbal confirmation (several times/week, high) — F2
4. Delete tasks that are cancelled or superseded to keep the list actionable (weekly, medium) — F3
5. Bookmark the "In Progress" filtered URL to use as a persistent daily status page (once, low ongoing effort) — F5

**Success Criteria:**
- Can produce a verbal team status update in under 2 minutes by scanning the filtered task list (supports PRD §7 "Task Completion Visibility")
- Task list reflects real team state within one working day — no stale items older than 24 hours without a status change (supports PRD §7 "Adoption")
- Can correct a task's details without needing to contact the original creator (PRD §6 Usability, F2 independence)
- All team members (100%) using TaskFlow within 2 weeks, eliminating the need for a parallel spreadsheet (PRD §7 "Adoption")

---

## Persona Relationships

| Interaction | PER-01 Marcus (Team Member) | PER-02 Priya (Team Lead) |
|-------------|----------------------------|--------------------------|
| Task creation | Creates the majority of tasks representing his own work | Creates tasks for team-wide items; reviews tasks created by Marcus |
| Status updates | Updates task status as work progresses | Occasionally updates status on behalf of team members after verbal check-in |
| Task deletion | Deletes tasks he created that are cancelled | Deletes stale or obsolete tasks across the shared list |
| List visibility | Consumes shared list to avoid duplicating effort | Consumes shared list to build a team status picture |
| Trust in data | Needs confidence that his entries persist (F4) | Needs confidence that the list is the current source of truth (F4) |

Both personas interact with the **same shared workspace** — there is no per-user isolation in v1. Changes made by either persona are immediately visible to the other.

---

## Feature-Persona Matrix

| Feature ID | Feature Name | PER-01 Marcus (Team Member) | PER-02 Priya (Team Lead) |
|------------|--------------|----------------------------|--------------------------|
| F0 | Task Creation | **Primary** — creates tasks for his own work daily | **Secondary** — creates shared or team-wide tasks |
| F1 | Task List View | **Primary** — scans list to track team status and own work | **Primary** — daily status review; primary coordination surface |
| F2 | Task Edit | **Primary** — updates status and details of his own tasks | **Primary** — corrects task details and updates status on behalf of team |
| F3 | Task Deletion | **Primary** — removes cancelled or erroneous tasks | **Primary** — actively curates list to remove stale items |
| F4 | Data Persistence | **Primary** — trusts that entries survive restarts | **Primary** — relies on list as authoritative source of truth |
| F5 | Status Filtering | **Secondary** — uses "In Progress" filter to focus work | **Primary** — bookmarks filtered URL as daily status dashboard |

**Legend:** Primary = core to this persona's daily workflow | Secondary = used but not central to their key tasks

---

*Document generated: 2026-07-01 | Derived from PRD-TaskFlow.md v1.0 and PROJECT.md v1.0*
