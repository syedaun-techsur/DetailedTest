# User Journey Maps — TaskFlow
**Project Acronym:** TaskFlow
**Version:** 1.0
**Date:** 2026-07-01
**Status:** Draft
**Related Personas:** PERSONAS-TaskFlow.md v1.0
**Related JTBD:** JTBD-TaskFlow.md v1.0
**Related PRD:** PRD-TaskFlow.md v1.0

---

## Journey Index

| JRN-ID | Persona | Scenario | Key JTBD | Stages |
|--------|---------|----------|----------|--------|
| JRN-01.1 | PER-01 Marcus Webb (Team Member) | Capturing a new task mid-flow | JTBD-01.1 | 5 |
| JRN-01.2 | PER-01 Marcus Webb (Team Member) | Morning orientation and status check | JTBD-01.2 | 5 |
| JRN-01.3 | PER-01 Marcus Webb (Team Member) | Progressing and trusting a status update | JTBD-01.3 | 5 |
| JRN-02.1 | PER-02 Priya Nair (Team Lead) | Daily team status review | JTBD-02.1 | 6 |
| JRN-02.2 | PER-02 Priya Nair (Team Lead) | Identifying and triaging stuck or overdue work | JTBD-02.2 | 5 |
| JRN-02.3 | PER-02 Priya Nair (Team Lead) | Curating the shared task list for accuracy | JTBD-02.3 | 5 |

---

## PER-01: Marcus Webb — Team Member

---

### JRN-01.1: Capturing a New Task Mid-Flow

**Persona:** PER-01 (Marcus Webb)

**Scenario:** Marcus is reviewing a pull request when a teammate drops a Slack message asking him to investigate a build failure. He doesn't want to lose context, forget the request, or pin yet another Slack message that will drift out of sight. He opens TaskFlow in a new tab and captures the item in seconds before returning to his code review.

**Related Jobs:** JTBD-01.1

---

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|-------|--------|------------|----------|---------|------------|-------------|
| Triggered | Reads Slack request, decides to log it before forgetting | Slack (external) | "I need to write this down somewhere permanent before I forget it" | Mildly anxious — context switching is costly | Slack pins drift out of sight within hours; no shared record | Clear mental trigger: TaskFlow = the place to log work items |
| Navigate | Opens new browser tab, types or bookmarks TaskFlow URL | Browser address bar | "How quickly can I get back to my PR after this?" | Neutral, slightly impatient | No shortcut or quick-add from Slack; has to switch apps entirely | Consider bookmarkable direct link to new-task form |
| Create | Clicks "New Task" button, types title in the text field | Task Creation form (F0) | "Do I need to fill in anything else right now? Probably just the title" | Focused | Uncertain if optional fields block submission | Form must make clear that only title is required; optional fields are visibly de-emphasized |
| Submit | Presses submit / save button | Task Creation form (F0), Task List (F1) | "Did it save? Is it showing up?" | Relieved if list updates immediately; anxious if no feedback | No visual confirmation = doubt that the task was saved | Immediate redirect to task list with new task highlighted or confirmed |
| Return | Switches back to PR tab, knows the task is safely logged | Browser tab (external) | "Good. I can focus again — it's in the system" | Calm, confident | Residual doubt if task list wasn't visibly updated | Persistent task list ensures next time Marcus checks, the item is exactly where he left it (F4) |

#### Key Moments

- **Decision Point:** Create stage — Marcus decides whether to fill optional fields now or just title. If the form feels mandatory, he may abandon and revert to Slack.
- **Risk of Abandonment:** Navigate stage — if TaskFlow takes more than 2 clicks to reach an entry form, the friction exceeds "Slack pin," and Marcus will not bother.
- **Delight Opportunity:** Submit stage — task appears instantly in the shared list; Marcus sees it immediately. Zero doubt about whether it saved.

#### Success Outcome

Marcus captures a new task from Slack context within 60 seconds and returns to his code review without losing flow — matching JTBD-01.1 success measure (first task created in ≤60 seconds with zero friction).

#### Feature Touchpoints

| Stage | Features |
|-------|----------|
| Navigate | F1 (Task List — entry point) |
| Create | F0 (Task Creation form) |
| Submit | F0 (Task Creation), F4 (Data Persistence), F1 (Task List confirmation) |
| Return | F4 (Data Persistence — trust that task persists) |

---

### JRN-01.2: Morning Orientation and Status Check

**Persona:** PER-01 (Marcus Webb)

**Scenario:** Marcus sits down at 9 AM to start his day. Before pulling up his IDE he wants a quick picture of what the team is working on so he knows what to pick up next and doesn't accidentally duplicate a teammate's work. He has two minutes before standup.

**Related Jobs:** JTBD-01.2

---

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|-------|--------|------------|----------|---------|------------|-------------|
| Arrive | Opens browser, navigates to TaskFlow | Task List page (F1) | "Let me see what changed since yesterday" | Neutral, slightly rushed (standup soon) | Has to remember the URL or find a bookmark | Bookmarkable URL (F5 filter URL) makes return visits instant |
| Orient | Scans the default task list — all tasks visible | Task List page (F1) | "There's a lot here — what's actually active?" | Mildly overwhelmed | Full list includes Done tasks that are visual noise | Default sort (newest first) helps, but Done items still clutter the view |
| Filter | Clicks "In Progress" filter tab | Status filter controls (F5) | "Now I can see what's actually being worked on" | Focused | One click required; filter must be easy to find and clearly labeled | Bookmarkable "In Progress" URL so tomorrow's check skips this step entirely |
| Scan | Reads task titles and statuses in filtered list | Task List (F1, F5 active) | "Is anyone already on the build-fix ticket? Good — I can take the API task" | Relieved, purposeful | No assignee field in v1 — Marcus can't tell who is doing what from title alone | Clear, descriptive task titles are the only signal; encourage title discipline |
| Complete | Closes task list, opens IDE with a mental plan for the day | Browser → IDE (external) | "Standup in 3 minutes and I know exactly what's happening" | Confident | Possible that task list is stale if teammates haven't updated since yesterday | Real-time updates deferred to v2; encourage team norm of updating status at day start |

#### Key Moments

- **Decision Point:** Filter stage — if filtering is more than 1 click, Marcus may skip it and scan the full noisy list, reducing accuracy of his mental model.
- **Risk of Abandonment:** Orient stage — if Done tasks dominate the list and no filter is obvious, Marcus may conclude the tool is too noisy and revert to Slack.
- **Delight Opportunity:** Scan stage — Marcus immediately spots that the most urgent task is already claimed and can confidently choose a different task, saving a "who's on X?" Slack message.

#### Success Outcome

Marcus identifies his own plan for the day within 10 seconds of loading the filtered list — matching JTBD-01.2 success measure (status of any active task readable within 10 seconds).

#### Feature Touchpoints

| Stage | Features |
|-------|----------|
| Arrive | F1 (Task List) |
| Orient | F1 (Task List — all tasks view) |
| Filter | F5 (Status Filtering) |
| Scan | F1 (Task List), F5 (In Progress filter active) |

---

### JRN-01.3: Progressing and Trusting a Status Update

**Persona:** PER-01 (Marcus Webb)

**Scenario:** Marcus has just merged a PR for a task that was "In Progress." He wants to mark it "Done" and start a new item. He also had a previous bad experience with a tool that lost his updates after a server restart — he needs to believe TaskFlow won't do that.

**Related Jobs:** JTBD-01.3

---

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|-------|--------|------------|----------|---------|------------|-------------|
| Identify | Finds the completed task in the "In Progress" filtered list | Task List (F1, F5) | "There it is — the build-fix task I finished this morning" | Focused | Scan time increases if task titles are generic or list is long | Good task titles + filter reduce scan time to seconds |
| Open Edit | Clicks the task row or edit button | Task List row → Edit form (F2) | "I just want to change the status — I hope I don't have to fill in everything again" | Slightly anxious | Fear that edit form will require re-entering all fields | Edit form pre-populated (F2) — Marcus sees his original data intact, changes only status |
| Update Status | Changes status dropdown from "In Progress" to "Done" | Task Edit form (F2) | "Two clicks and it's done — that's what I want" | Satisfied | Dropdown may be hard to hit on a small screen (not a concern in v1 internal web use) | Inline status toggle directly from task list row would reduce clicks to 1 |
| Save | Clicks Save button | Task Edit form (F2), Task List (F1) | "Did it save? Will this still be here tomorrow after a restart?" | Anxious momentarily, then relieved | Previous tool experiences erode trust in persistence | Immediate return to list showing updated status; implicit reassurance that it's in the DB (F4) |
| Verify | Scrolls through task list, sees task now shows "Done" status | Task List (F1) | "Good. The whole team can see it's done now" | Confident, validated | No explicit "saved to database" indicator — trust is implicit | Add a subtle save-confirmation toast or badge so persistence is communicated visually |

#### Key Moments

- **Decision Point:** Open Edit stage — if the edit flow requires Marcus to navigate to a separate page with no back button, he will lose context and feel the tool is heavyweight.
- **Risk of Abandonment:** Save stage — if there is no visual confirmation that the change persisted, Marcus will refresh the page to verify, and if it's still correct he gains trust incrementally. If not, trust collapses.
- **Delight Opportunity:** Verify stage — Marcus sees the task marked Done in the shared list and knows the team can see his progress without him sending a Slack message. Silent broadcast of done work.

#### Success Outcome

Marcus updates a task's status in 2 clicks from the task list and sees the change reflected immediately in the shared view — matching JTBD-01.3 success measure (status updated in ≤2 clicks; zero data loss over 30 days).

#### Feature Touchpoints

| Stage | Features |
|-------|----------|
| Identify | F1 (Task List), F5 (In Progress filter) |
| Open Edit | F2 (Task Edit — row click or edit button) |
| Update Status | F2 (Task Edit form — status dropdown) |
| Save | F2 (Task Edit), F4 (Data Persistence) |
| Verify | F1 (Task List — updated status visible) |

---

## PER-02: Priya Nair — Team Lead / Coordinator

---

### JRN-02.1: Daily Team Status Review

**Persona:** PER-02 (Priya Nair)

**Scenario:** It is 8:45 AM on a Thursday. Priya has a department head check-in at 10 AM and needs to mentally prepare a team status summary. She opens the task list, applies her saved "In Progress" filter, and scans the active work to understand what is in flight, what is at risk, and whether anything needs her intervention before the check-in.

**Related Jobs:** JTBD-02.1

---

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|-------|--------|------------|----------|---------|------------|-------------|
| Arrive | Opens browser bookmark for TaskFlow "In Progress" filtered URL | Task List (F1, F5 — In Progress pre-filtered) | "Let me see what everyone has in flight before the check-in" | Purposeful, slightly pressured | If bookmark doesn't preserve filter state, must reapply filter manually | Bookmarkable filter URL (F5) eliminates setup friction; Priya's daily ritual is one click |
| Scan | Reads through "In Progress" task titles and due dates | Task List (F1, F5 active) | "Six active tasks. Three look fine. Two have due dates from last week — those are the ones to probe" | Alert, analytical | No assignee in v1 — can't tell at a glance which team member owns each task | Descriptive task titles (team convention) partially compensate; v1 limitation accepted |
| Spot Risk | Identifies two tasks with past due dates in "In Progress" status | Task List — due date column (F1) | "These two should have moved to Done or been updated. I need to follow up with whoever is on them" | Concerned, investigative | Due date visibility depends on list column rendering; must be visible without clicking in | Due date must render on each row in the task list (F1 design requirement) |
| Assess | Reviews remaining "In Progress" tasks for completeness | Task List (F1, F5) | "The other four look reasonable. I can speak to these in the check-in" | Relieved, confident | No "last updated" timestamp in v1 makes it hard to tell if a task was recently touched | v2 candidate: show updated_at on task row |
| Prepare | Mentally composes a verbal status summary based on what she has read | Task List (passive — no clicks) | "I have what I need. Six in flight, two at risk. Two minutes and I'm ready" | Confident | Must hold status picture in memory; no export or summary view | v2 candidate: printable or copy-pasteable summary of filtered task list |
| Act | Sends a Slack message to the two at-risk task contributors before the check-in | Slack (external) | "A quick heads-up will prevent a surprise in the meeting" | Proactive | Identifying which person to contact requires knowing who created the task (no assignee in v1) | Task titles that include the owner's name (team convention) bridge the v1 gap |

#### Key Moments

- **Decision Point:** Scan stage — Priya decides which tasks need follow-up and which are on track. Accuracy of this decision depends entirely on task data being current; stale data produces false confidence.
- **Risk of Abandonment:** Arrive stage — if the bookmarked URL does not preserve the filter, Priya must reapply "In Progress" manually every day; over time this friction erodes the habit.
- **Delight Opportunity:** Spot Risk stage — Priya identifies an overdue task in under 30 seconds and heads off a stakeholder surprise. The tool did in seconds what previously required a morning of Slack chasing.

#### Success Outcome

Priya produces a verbal team status update in under 2 minutes from opening the filtered view — matching JTBD-02.1 success measure, without sending a single "what's the status of X?" Slack message.

#### Feature Touchpoints

| Stage | Features |
|-------|----------|
| Arrive | F5 (Bookmarkable filter URL), F1 (Task List) |
| Scan | F1 (Task List — title, status, due date visible inline) |
| Spot Risk | F1 (due date column), F5 (In Progress filter isolates relevant tasks) |
| Assess | F1 (Task List), F4 (Data Persistence — trust that data is current) |
| Prepare | F1 (Task List passive read) |

---

### JRN-02.2: Identifying and Triaging Stuck or Overdue Work

**Persona:** PER-02 (Priya Nair)

**Scenario:** It is Monday morning after a bank holiday weekend. Priya is concerned some tasks may have missed their Friday due date without a status update. She opens the filtered "In Progress" list specifically looking for tasks with past due dates that are still active, so she can proactively follow up before her team standup at 9:30 AM.

**Related Jobs:** JTBD-02.2

---

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|-------|--------|------------|----------|---------|------------|-------------|
| Focus | Opens TaskFlow directly to "In Progress" filter via bookmark | Task List (F1, F5 — In Progress) | "I expect at least one or two tasks to be overdue after the long weekend" | Alert, bracing for bad news | Filter must have survived as a usable bookmark; if not, extra step to reapply | URL-persisted filter state (F5) is load-bearing for this persona's workflow |
| Scan Due Dates | Reads due date column across all "In Progress" tasks | Task List — due date column (F1) | "That one was due Thursday. That one was due last Tuesday. Both are still In Progress" | Concerned | Due dates only show if set — tasks without due dates are ambiguous (could be fine or forgotten) | Team discipline: set due dates on all tasks; or v2 adds a "days in current status" indicator |
| Identify Overdue | Mentally flags 2–3 tasks with past due dates still showing "In Progress" | Task List (F1, F5) | "These haven't moved since before the holiday. That's a problem or a missed update" | Frustrated by ambiguity | Can't tell if the task is "done but not updated" vs. "genuinely blocked" | v1 workaround: Priya edits the task to add a note in the description; v2: comment thread |
| Validate | Clicks into one overdue task to read its description for any context | Task detail / Edit form (F2) | "Is there any information in here that tells me what's happening?" | Investigative | Description field may be blank if contributor didn't fill it out | Encourage description hygiene; F2 pre-populated edit makes it easy for Priya to add context |
| Follow Up | Returns to task list and notes which team members to contact at standup | Task List (F1) | "I'll flag these two at standup — no Slack message needed yet" | Resolved, action-oriented | No assignee makes identifying the right person to contact manual | Team naming convention in task titles carries this gap in v1 |

#### Key Moments

- **Decision Point:** Identify Overdue stage — Priya decides whether an overdue item is "done but not updated" (quick fix) or "genuinely stuck" (needs escalation). This decision is based only on task description and title; missing context forces a chat interruption.
- **Risk of Abandonment:** Scan Due Dates stage — if due dates are not visible inline on each task row, Priya must click into each task individually. If this takes more than a few clicks, she reverts to "just ask at standup" and stops relying on the tool.
- **Delight Opportunity:** Focus stage — the bookmark drops Priya directly into the risk-relevant filtered view. The tool does the filtering work that previously required manual spreadsheet scanning.

#### Success Outcome

Priya identifies all overdue "In Progress" tasks within 60 seconds of opening the filtered view — matching JTBD-02.2 success measure, without requiring input from any team member.

#### Feature Touchpoints

| Stage | Features |
|-------|----------|
| Focus | F5 (Bookmarkable filter URL), F1 (Task List) |
| Scan Due Dates | F1 (due date visible inline on each row) |
| Identify Overdue | F1 (Task List), F5 (In Progress filter active) |
| Validate | F2 (Task Edit / detail view — read description) |
| Follow Up | F1 (Task List) |

---

### JRN-02.3: Curating the Shared Task List for Accuracy

**Persona:** PER-02 (Priya Nair)

**Scenario:** Friday afternoon. Priya does a weekly list cleanup. She finds three problems: one task has a vague title entered by a teammate ("misc stuff"), one task was verbally confirmed as done in standup but never marked Done, and one task was cancelled two weeks ago but is still in the list. She wants to fix all three without messaging anyone, then bookmark the clean list for next week.

**Related Jobs:** JTBD-02.3

---

#### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|-------|--------|------------|----------|---------|------------|-------------|
| Survey | Removes "In Progress" filter; scans full task list in default view | Task List (F1 — All tasks) | "Let me see everything, including Done tasks, and clean up the clutter" | Methodical, slightly resigned | Done tasks mixed with active ones make the full list noisy; no archive view | Consider a "Done" filter or separate Done section as a v1.1 improvement |
| Edit Title | Clicks edit on the vague-titled task, updates title to something descriptive | Task Edit form (F2) | "If I can correct this now, everyone benefits — I don't need the original author's permission" | Empowered | Edit form pre-populated (F2) — only changes the title, saves immediately | No-permission-check edit is core to v1 design; makes Priya's curation role frictionless |
| Update Status | Opens the "verbally done" task, changes status from "In Progress" to "Done" | Task Edit form (F2) | "This was confirmed done on Wednesday — it just needs the list to reflect reality" | Pragmatic | Without an activity log (v2+), there's no record that Priya changed it | Pre-populated edit form means this is a 2-click status change; minimal overhead |
| Delete Stale Task | Clicks delete on the cancelled task; reads confirmation dialog; confirms deletion | Task List row → Delete confirmation (F3) | "Are you sure? Yes I'm sure — this was cancelled 2 weeks ago" | Decisive | Confirmation dialog is necessary friction; acceptable given irreversibility | Confirmation dialog text should state clearly what will be deleted and that it's permanent |
| Verify Clean List | Returns to task list, scans to confirm all three changes took effect | Task List (F1) | "Good — title fixed, status updated, stale task gone. The list reflects reality now" | Satisfied, accomplished | No "last edited by" indicator means Priya can't confirm which changes are hers vs others | v2: updated_at + editor attribution; v1: Priya trusts her own actions |

#### Key Moments

- **Decision Point:** Delete Stale Task stage — the confirmation dialog is the critical gate. If it is ambiguous (unclear what will be deleted), Priya may cancel unnecessarily. If it is absent, accidental mass deletion is a real risk.
- **Risk of Abandonment:** Survey stage — if Priya cannot easily see the full task list without filter friction, she will not complete the weekly cleanup, and the list will accumulate noise over time.
- **Delight Opportunity:** Edit Title stage — Priya corrects a teammate's vague task title without any permission prompt or workflow friction. The tool implicitly says "you are trusted to maintain this list." This matches the v1 design decision (no per-user ownership).

#### Success Outcome

Priya corrects all three list problems in a single session without sending a chat message or waiting for anyone — matching JTBD-02.3 success measure (shared list maintained as single source of truth; 100% team adoption within 2 weeks by establishing Priya as an active curator).

#### Feature Touchpoints

| Stage | Features |
|-------|----------|
| Survey | F1 (Task List — All tasks view), F5 (filter removed to show all) |
| Edit Title | F2 (Task Edit — pre-populated form) |
| Update Status | F2 (Task Edit — status dropdown) |
| Delete Stale Task | F3 (Task Deletion — confirmation dialog), F4 (Data Persistence — immediate removal from DB) |
| Verify Clean List | F1 (Task List — updated state visible to all) |

---

## Cross-Journey Patterns

### Common Pain Points Across Journeys

- **No assignee / ownership in v1 (JRN-01.2, JRN-02.1, JRN-02.2, JRN-02.3):** Every persona eventually wants to know who is responsible for a task. The v1 workaround is task-title naming conventions. This is the strongest candidate for a v2 feature.
- **Trust in persistence after save (JRN-01.3, JRN-02.3):** Both Marcus and Priya experience a moment of doubt after saving — "did that actually stick?" A subtle save-confirmation toast or visual indicator would close this gap without adding UI complexity.
- **Stale data when teammates don't update (JRN-01.2, JRN-02.1, JRN-02.2):** The tool's utility is directly proportional to how consistently the team updates task status. Real-time updates (v2) would solve this structurally; in v1, team norms and lightweight daily update habits are the mitigation.
- **No last-updated timestamp visible on task rows (JRN-02.1, JRN-02.2):** Priya cannot distinguish "recently active In Progress" from "forgotten In Progress" without clicking into each task. A `updated_at` display on the task row is a low-cost v1.1 improvement with high value for the lead persona.

### Shared Opportunities

- **Bookmarkable filtered URL (F5) is load-bearing for both personas:** Marcus can bookmark his "In Progress" morning check; Priya can bookmark her daily status view. This feature reduces per-session setup from 2 steps to 0 and directly supports habit formation and adoption.
- **Pre-populated edit form (F2) serves both personas equally:** Marcus needs it to do a 1-field status change without re-entering data. Priya needs it to correct a single title field without disrupting other values. The same design decision serves both use cases identically.
- **Empty state clarity (F1):** When a filter returns no results, both personas need a clear empty state message — not a confusing blank page. This is a low-effort, high-trust detail that signals the tool is working as intended.

### Converging Stages

Both personas converge on the **Task List (F1)** as the primary surface for every journey. Every journey starts and ends at the task list. This makes F1's design — column layout, status badge visibility, due date display, filter controls, empty state — the highest-leverage design investment in the entire product.

---

## Journey-to-JTBD Traceability

| JRN-ID | Stage | JTBD-ID | Expected Outcome |
|--------|-------|---------|-----------------|
| JRN-01.1 | Create | JTBD-01.1 | Task created with title only, no account setup, in ≤60 seconds |
| JRN-01.1 | Submit | JTBD-01.1 | Task immediately visible in shared list; F4 confirms persistence |
| JRN-01.2 | Filter | JTBD-01.2 | "In Progress" filter reduces noise in 1 click |
| JRN-01.2 | Scan | JTBD-01.2 | Status of any active task readable within 10 seconds of loading |
| JRN-01.3 | Open Edit | JTBD-01.3 | Pre-populated form confirms prior data is intact (F4 trust signal) |
| JRN-01.3 | Save | JTBD-01.3 | Status update persisted to DB in ≤2 clicks; survives restart |
| JRN-02.1 | Arrive | JTBD-02.1 | Bookmarked filter URL opens pre-filtered In Progress view instantly |
| JRN-02.1 | Scan | JTBD-02.1 | Title, status, due date all visible inline — no clicks into tasks |
| JRN-02.1 | Spot Risk | JTBD-02.1 | Overdue tasks identifiable in <30 seconds via due date column |
| JRN-02.2 | Scan Due Dates | JTBD-02.2 | Due dates visible on task rows without opening each task |
| JRN-02.2 | Identify Overdue | JTBD-02.2 | All overdue In Progress tasks surfaced within 60 seconds |
| JRN-02.3 | Edit Title | JTBD-02.3 | Any field editable by any team member; no permission prompt |
| JRN-02.3 | Update Status | JTBD-02.3 | Status corrected on behalf of team member in ≤2 clicks |
| JRN-02.3 | Delete Stale Task | JTBD-02.3 | Task permanently removed with confirmation; disappears for all viewers |
| JRN-02.3 | Verify Clean List | JTBD-02.3 | Updated list reflects reality; serves as single source of truth |

---

*Document generated: 2026-07-01 | Derived from PERSONAS-TaskFlow.md v1.0, JTBD-TaskFlow.md v1.0, PRD-TaskFlow.md v1.0*
