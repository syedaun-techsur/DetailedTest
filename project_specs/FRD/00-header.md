# Functional Requirements Document — TaskFlow

**Project Acronym:** TaskFlow
**Version:** 1.0
**Date:** 2026-07-01
**Status:** Draft
**Based On:** PRD-TaskFlow.md v1.0, PROJECT.md v1.0

---

## Scope

This document specifies the complete functional behavior of TaskFlow v1 — a lightweight, shared task-tracking web application for small internal teams of 2–15 people. It covers all six PRD features (F0–F5), the relational database schema, the REST API endpoint catalog, the cross-feature error catalog, and external integration points. Downstream agents (code generators, test generators, reviewers) should treat this document as the single authoritative specification for what the system must do.

Authentication, multi-tenancy, real-time updates, file attachments, and task assignment are **explicitly out of scope for v1** and must not be implemented.

---

## Conventions

- **Feature IDs:** F00–F05 map 1:1 to PRD features F0–F5.
- **Field types:** SQL-style types used throughout (VARCHAR, TEXT, TIMESTAMP, etc.).
- **Required vs Optional:** `(required)` and `(optional)` annotations on all input fields.
- **HTTP verbs:** Standard REST semantics — GET (read), POST (create), PUT/PATCH (update), DELETE (remove).
- **Error codes:** Uppercase snake-case strings (e.g., `TITLE_REQUIRED`).
- **Enum values:** Status values are always one of exactly: `To Do`, `In Progress`, `Done` (display form) or `todo`, `in_progress`, `done` (URL/query-param form).
- **Timestamps:** All timestamps stored and returned in UTC ISO 8601 format (`YYYY-MM-DDTHH:MM:SSZ`).
- **Date-only fields:** `due_date` stored as `DATE` (no time component). Accepted input format: `YYYY-MM-DD`.
- **Cross-references:** `see F03 §Process` means refer to feature chunk F03, Process section.

---

## Master Table of Contents

| Section | File | Description |
|---------|------|-------------|
| F00 | `F00-task-creation.md` | Task Creation |
| F01 | `F01-task-list-view.md` | Task List View |
| F02 | `F02-task-edit.md` | Task Edit |
| F03 | `F03-task-deletion.md` | Task Deletion |
| F04 | `F04-data-persistence.md` | Data Persistence |
| F05 | `F05-status-filtering.md` | Status Filtering |
| Y0 | `Y0-schema.md` | Database Schema (DDL) |
| Y1 | `Y1-api.md` | REST API Endpoint Catalog |
| Y2 | `Y2-errors.md` | Cross-Feature Error Catalog |
| Y3 | `Y3-integrations.md` | External Integration Points |

---

## Cross-Cutting Terminology

- **Task:** The core domain entity. Represents a unit of work with a title, optional description, status, optional due date, and system-managed timestamps.
- **Shared Workspace:** The single, globally shared context in which all tasks exist. There is no per-user, per-team, or per-project isolation in v1.
- **Status:** An enum field on a task. Valid values: `To Do` (default), `In Progress`, `Done`.
- **Status slug:** The URL-safe, lowercase version of a status value used in query parameters: `todo`, `in_progress`, `done`.
- **Due Date:** An optional calendar date (no time) by which a task is intended to be completed.
- **Task List:** The primary view — a server-rendered page showing all tasks in the shared workspace, optionally filtered by status.
- **Task Detail / Edit View:** A server-rendered page showing a single task's full details and providing an edit form.
- **Confirmation Step:** A user-facing dialog or dedicated page that asks the user to explicitly confirm a destructive action (deletion) before it is executed.
- **ORM:** Object-Relational Mapper. TaskFlow uses Sequelize (Node.js) or Django ORM (Python) to issue parameterized queries; raw SQL is not used directly.
- **Migration:** A versioned, repeatable script that creates or alters the database schema. Required for initial setup and future schema changes.
- **Empty State:** The UI condition where a list contains zero items, requiring a specific message to be displayed instead of a blank list.

---

## Non-Functional Constraints (Summary)

| Category | Requirement |
|----------|-------------|
| Performance | Task list page loads ≤ 2 seconds on single-server with up to 500 tasks |
| Reliability | Zero data loss on server restart; DB writes synchronous and confirmed before HTTP response |
| Security | XSS prevention via output escaping; SQL injection prevention via ORM parameterized queries |
| Accessibility | Form inputs have `<label>` elements; status badges meet WCAG AA color contrast |
| Browser Support | Chrome, Firefox, Safari, Edge (current versions) |
| Usability | Core CRUD completable in ≤ 3 clicks from the task list |
| Portability | Runnable via single command after initial setup (`npm start` or `python manage.py runserver`) |

---

*End of header — see feature chunks F00–F05 and cross-feature chunks Y0–Y3 for full specification.*
