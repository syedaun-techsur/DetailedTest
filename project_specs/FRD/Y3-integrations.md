---

## Y3: External Integration Points

**Scope:** All external systems and dependencies that TaskFlow v1 interacts with at runtime or development time. TaskFlow v1 is intentionally minimal and has very few external integration points.

---

### Integration Summary

| # | System | Type | Required | Scope |
|---|--------|------|----------|-------|
| 1 | Relational Database (PostgreSQL / SQLite) | Data store | Yes | All features (runtime) |
| 2 | Node.js / Python Runtime | Execution environment | Yes | All features (runtime) |
| 3 | ORM (Sequelize / Django ORM) | Query abstraction layer | Yes | All features (development + runtime) |
| 4 | Browser (client-side rendering) | User interface | Yes | All features (client) |

There are **no third-party external service calls** in TaskFlow v1. No email, no push notifications, no external authentication providers, no analytics services, no CDNs, no object storage.

---

### Integration 1: Relational Database

**System:** PostgreSQL 14+ (production) / SQLite 3.x (development)
**Purpose:** Persistent storage of all task data
**Protocol:** TCP (PostgreSQL) / local file (SQLite)
**Configuration:** Via `DATABASE_URL` environment variable

| Property | Value |
|----------|-------|
| Connection method | ORM connection pool (Sequelize) or Django DB settings |
| Connection string format | `postgres://user:pass@host:port/dbname` or `sqlite:///path/to/file.db` |
| Max connections (PostgreSQL) | 5–10 (suitable for small team, single server) |
| Auth mechanism | Username + password (PostgreSQL); none (SQLite) |
| SSL | Recommended for production PostgreSQL; not required for SQLite |
| Failure behavior | HTTP 503 / 500; full error logged server-side |

**Dependency:** The application process cannot start successfully if `DATABASE_URL` is not set and valid. A startup health check should verify DB connectivity before accepting requests.

---

### Integration 2: Node.js / Python Runtime

**System:** Node.js 18+ (if Express) or Python 3.10+ (if Django)
**Purpose:** Application execution environment
**Type:** Process runtime — not an external service

| Property | Value |
|----------|-------|
| Node.js version | 18 LTS or higher |
| Python version | 3.10 or higher |
| Package manager | npm (Node) / pip + requirements.txt or poetry (Python) |
| Startup command | `npm start` (Node) / `python manage.py runserver` (Django) |
| Port | Configurable via `PORT` environment variable; default 3000 (Node) or 8000 (Django) |

---

### Integration 3: ORM (Sequelize / Django ORM)

**System:** Sequelize 6+ (Node) or Django ORM (built-in with Django 4+)
**Purpose:** Parameterized query generation, connection pooling, migration management
**Type:** Library dependency — not an external service

| Property | Value |
|----------|-------|
| Version | Sequelize 6.x (Node) / Django 4.x built-in ORM |
| Migration tool | sequelize-cli (Node) / `python manage.py migrate` (Django) |
| Dialect support | postgres, sqlite (both ORMs) |
| Raw SQL | Not used; all queries via ORM methods |
| Security benefit | Parameterized queries prevent SQL injection by default |

---

### Integration 4: Browser (Client-side)

**System:** Web browser (Chrome, Firefox, Safari, Edge — current versions)
**Purpose:** Renders server-generated HTML; handles form submissions
**Type:** Client — not a service integration

| Property | Value |
|----------|-------|
| Rendering model | Server-rendered HTML (no SPA framework required) |
| JavaScript requirement | Minimal; confirmation dialogs may use `window.confirm()` or a lightweight JS modal |
| CSS dependencies | None mandated; standard CSS is sufficient |
| Form method support | HTML forms only support GET/POST; PUT/DELETE require `_method` override in form body |
| Cookies / Sessions | Not used in v1 (no authentication, no server-side sessions) |

---

### Out-of-Scope Integrations (v2+)

The following integrations are explicitly NOT part of v1 and must not be implemented:

- **Email / Notifications:** No email delivery, no webhook, no Slack/Teams integration.
- **Authentication Provider:** No OAuth, SAML, LDAP, or SSO.
- **Object Storage:** No S3, GCS, or local file storage for attachments.
- **Analytics / Monitoring:** No Segment, Mixpanel, Sentry, Datadog, etc. (basic server logging is acceptable).
- **CDN:** No CDN for static assets (single-server deployment serves static files directly).
- **WebSockets / Server-Sent Events:** No real-time push; clients refresh the page to see updates.

---

*End of integration points document.*
