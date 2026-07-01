---

## Y0: Database Schema

**Scope:** Full DDL for all TaskFlow v1 entities. TaskFlow v1 has a single domain table: `tasks`. No user, session, or tenant tables exist (no authentication, no multi-tenancy).

**Database Support:**
- **Production:** PostgreSQL 14+
- **Development/Testing:** SQLite 3.x (with ORM-managed dialect differences)

The DDL below is written in PostgreSQL syntax. ORM migration tools (Sequelize / Django ORM) generate the equivalent SQLite DDL automatically when the `DATABASE_URL` points to a SQLite file.

---

### tasks Table

```sql
-- PostgreSQL DDL

CREATE TABLE tasks (
    id          SERIAL PRIMARY KEY,
    title       VARCHAR(255)  NOT NULL,
    description TEXT          NULL,
    status      VARCHAR(20)   NOT NULL DEFAULT 'To Do'
                    CHECK (status IN ('To Do', 'In Progress', 'Done')),
    due_date    DATE          NULL,
    created_at  TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

---

### Column Definitions

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| `id` | SERIAL (PostgreSQL) / INTEGER AUTOINCREMENT (SQLite) | NOT NULL | auto | Primary Key | System-assigned; never set by application code |
| `title` | VARCHAR(255) | NOT NULL | — | ≤ 255 chars | Also validated at application layer (F00, F02) |
| `description` | TEXT | NULL | NULL | — | Stored as NULL when not provided; no length limit at DB layer |
| `status` | VARCHAR(20) | NOT NULL | `'To Do'` | CHECK: one of `'To Do'`, `'In Progress'`, `'Done'` | DB-level constraint mirrors application-level enum |
| `due_date` | DATE | NULL | NULL | Valid calendar date | No time component; NULL when not provided |
| `created_at` | TIMESTAMP WITH TIME ZONE | NOT NULL | `NOW()` | — | Set at INSERT; never updated |
| `updated_at` | TIMESTAMP WITH TIME ZONE | NOT NULL | `NOW()` | — | Set at INSERT; updated on every UPDATE |

---

### Indexes

```sql
-- Default index on primary key (created automatically)
-- Additional indexes for query performance

CREATE INDEX idx_tasks_status ON tasks (status);
CREATE INDEX idx_tasks_created_at ON tasks (created_at DESC);
```

- `idx_tasks_status`: Supports `WHERE status = $1` queries used by F01/F05 status filtering.
- `idx_tasks_created_at`: Supports `ORDER BY created_at DESC` default sort on the task list.

---

### Migration Script

The schema must be created and managed via ORM migration tools, not via manually executed SQL. The above DDL represents the target schema state after the initial migration runs.

**Node.js / Sequelize:**
```bash
npx sequelize-cli db:migrate
```

**Python / Django:**
```bash
python manage.py migrate
```

**Environment Variable:**
```
DATABASE_URL=postgres://user:password@host:5432/taskflow   # PostgreSQL (production)
DATABASE_URL=sqlite:///./taskflow.db                        # SQLite (local dev)
```

---

### Data Constraints Summary

- A `tasks` row with `title = NULL` must never be inserted (enforced at both application and DB layers).
- A `tasks` row with `status` outside the allowed enum must never be inserted (enforced at both application and DB layers).
- `id` values are never reused after deletion (SERIAL/AUTOINCREMENT).
- No foreign keys exist in v1 (single-table design).
- No soft-delete column (`deleted_at`, `is_deleted`) exists in v1 — all deletes are hard deletes.

---

### Future Schema Notes (out of scope for v1)

The following columns/tables are candidates for v2+ but must NOT be added in v1:
- `assigned_to` (FK to a users table) — requires authentication
- `deleted_at` (soft-delete) — v1.1 candidate
- `project_id` / `workspace_id` — multi-tenancy, v2+
- `comments` table — v2+
