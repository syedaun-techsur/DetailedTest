# UX Mockup — TaskFlow

**Project:** TaskFlow
**Generated:** 2026-07-01
**Based on:** UserStories-TaskFlow.md v1.0, PRD-TaskFlow.md v1.0, FRD-TaskFlow.md v1.0, JOURNEYS-TaskFlow.md v1.0, PROJECT.md v1.0

---

## Overview

TaskFlow is a minimal shared task-tracking web application for small internal teams (2–15 people). The UX is intentionally conventional: server-rendered HTML pages, standard form interactions, and no client-side routing. The product has one primary surface — the **Task List** — and two secondary surfaces: **Task Create Form** and **Task Edit Form**. Every user journey begins and ends at the Task List.

### Design Principles

1. **Task List is the hub.** Every flow returns to `/tasks`. All CRUD actions are reachable within 3 clicks from the list.
2. **Minimal friction for creation.** Only the title is required. Optional fields are visually de-emphasized so users (esp. Marcus) can capture a task in seconds without being blocked by optional fields.
3. **Pre-populated edits.** The edit form always shows current values — users change only what they need without re-entering existing data.
4. **Mandatory confirmation for deletion.** A confirmation gate (modal dialog or dedicated page) is non-negotiable before permanent deletion. The message names the task title and states "This action cannot be undone."
5. **Bookmarkable filter state.** The active status filter is encoded in the URL (`?status=in_progress`) so Priya can bookmark her morning view.
6. **All states designed.** Default, loading, success, error, and empty states are explicitly designed for every screen.
7. **WCAG AA compliance.** Status badges have sufficient color contrast; all form inputs have associated `<label>` elements; keyboard navigation works throughout.

### Screen Inventory

| Screen | Route | User Stories |
|--------|-------|-------------|
| Task List | `GET /tasks` | US-1.1, US-1.2, US-1.3, US-5.1, US-5.2, US-5.3, US-5.4 |
| Task Create Form | `GET /tasks/new` | US-0.1, US-0.2, US-0.3 |
| Task Edit Form | `GET /tasks/:id/edit` | US-2.1, US-2.2, US-2.3, US-2.4 |
| Delete Confirmation | (modal/dialog on Task List) | US-3.1, US-3.2, US-3.3 |
| Error Pages | (400, 404, 500) | US-3.3, US-4.3 |

### Flow Inventory

| Flow | Trigger | User Stories |
|------|---------|-------------|
| Flow-00: Create Task | Click "New Task" from Task List | US-0.1, US-0.2, US-0.3 |
| Flow-01: View & Filter Task List | Load `/tasks` | US-1.1, US-1.2, US-1.3, US-5.1–5.4 |
| Flow-02: Edit Task | Click task row or Edit button | US-2.1, US-2.2, US-2.3, US-2.4 |
| Flow-03: Delete Task | Click Delete button | US-3.1, US-3.2, US-3.3 |
