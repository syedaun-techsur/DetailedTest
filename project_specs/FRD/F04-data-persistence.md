---

## F04: Data Persistence

**Description:** This feature defines the infrastructure requirement that all task data is stored in a relational database and survives application server restarts. It is not a user-facing feature but an architectural constraint that underpins the correctness of all other features. In-memory or session-scoped storage is explicitly prohibited. Every create, update, and delete operation must write synchronously to the database, and the HTTP response must not be sent until the database write has been confirmed. The schema must be reproducible from scratch via migration scripts or an ORM migration command.

---

### Terminology

- **Relational Database:** A structured data store using tables, rows, and columns, supporting SQL queries and ACID transactions. PostgreSQL is the production default; SQLite is permitted for local development and testing only.
- **In-Memory Store:** Any data structure (JavaScript object, Python dict, array, etc.) that holds task data in application process memory. Explicitly prohibited for task records in v1.
- **Migration:** A versioned, idempotent script that creates or modifies the database schema. Must be runnable from the command line to set up a fresh database.
- **Synchronous Write:** A database write operation that completes (and is acknowledged by the DB) before the HTTP response is sent to the client. The server must not respond with a success redirect before the INSERT/UPDATE/DELETE has been committed.
- **Schema Drift:** A condition where the live database schema diverges from the version-controlled migration scripts. Prevented by using ORM migrations from project inception.

---

### Sub-features

- Define and version-control the `tasks` table schema
- Provide a migration command to create the schema from scratch
- Ensure all writes (create, update, delete) are synchronous and confirmed before response
- Ensure task list reads directly from the database on every request (no caching layer)
- Support PostgreSQL (production) and SQLite (local dev/test) without code changes
- Survive server restarts without data loss

---

### Process

1. **Initial setup:** Developer runs the migration command (e.g., `npx sequelize-cli db:migrate` or `python manage.py migrate`) against a new or existing database. The `tasks` table is created if it does not exist.
2. **Runtime — write operations:** On every task create, update, or delete:
   a. Application issues the appropriate SQL statement via ORM.
   b. Application awaits ORM promise/async resolution (confirming DB write).
   c. Only after confirmed write does the application issue the HTTP redirect or response.
3. **Runtime — read operations:** On every task list or task detail request, the application queries the database directly. No in-memory cache, no session storage.
4. **Server restart:** Application process stops and restarts. No task data is lost because all data resides in the database. The application reconnects to the database on startup.
5. **Schema changes (future):** New migrations are added and run. Rollback scripts are provided for each migration.

---

### Inputs

- No direct user inputs for this feature.
- Indirectly, all write operations from F00, F02, F03 feed into this persistence layer.
- Database connection parameters (DSN/URL) provided via environment variable (e.g., `DATABASE_URL`).

---

### Outputs

- `tasks` table exists in the database after migration is run.
- All task data written by F00/F02/F03 is durable across server restarts.
- Database connection error surfaces as HTTP 500 to users (see §Error States).

---

### Validation Rules

- **No in-memory storage:** Any implementation using a module-level array, object, or Map to hold task records between requests is non-compliant.
- **Synchronous writes:** Any implementation that sends an HTTP 2xx/3xx before `await db.save()` (or ORM equivalent) resolves is non-compliant.
- **Migration required:** Schema must be created via a migration tool, not via a one-off `CREATE TABLE` command run manually. The migration file must be version-controlled.
- **SQLite restriction:** SQLite may be used for local development and testing. PostgreSQL must be used for any shared or production deployment. The application must support switching between the two via environment variable only (no code changes required).
- **No raw SQL:** All queries must use ORM methods or parameterized queries. Raw string-concatenated SQL is non-compliant.

---

### Error States

| Scenario | HTTP Status | Error Code | User-Facing Message |
|----------|-------------|------------|---------------------|
| Database connection fails on startup | N/A (process crash or health check failure) | `DB_CONNECTION_ERROR` | "Unable to connect to the database." (logged; health endpoint returns 503) |
| Database write fails mid-request | 500 | `DB_WRITE_ERROR` | "Unable to save the task. Please try again." |
| Database read fails mid-request | 500 | `DB_READ_ERROR` | "Unable to load tasks. Please try again." |
| Migration not run (table missing) | 500 | `DB_SCHEMA_MISSING` | "Database schema not initialized. Run migrations." (logged; not shown in prod) |

---

### API Surface (this feature)

This feature has no dedicated API endpoints. It is a cross-cutting infrastructure concern enforced by the implementation of F00, F01, F02, and F03.

---

### Schema Surface (this feature)

Defines the canonical `tasks` table schema. Full DDL: see `Y0-schema.md` §tasks table.

Key column constraints:
- `id`: SERIAL / AUTOINCREMENT primary key. Never NULL.
- `title`: VARCHAR(255) NOT NULL. Enforced at DB level in addition to application validation.
- `status`: VARCHAR(20) NOT NULL DEFAULT 'To Do'. CHECK constraint: `status IN ('To Do', 'In Progress', 'Done')`.
- `created_at` / `updated_at`: TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(). `created_at` never updated after insert; `updated_at` always updated on each UPDATE.
