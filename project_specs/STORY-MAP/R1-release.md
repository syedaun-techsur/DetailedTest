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
