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
