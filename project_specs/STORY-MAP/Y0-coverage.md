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
