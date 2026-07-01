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
