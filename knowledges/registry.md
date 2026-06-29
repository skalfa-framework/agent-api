# Skalfa Knowledge Registry (Backend)

Before creating any implementation plan (`Stage 1 — Planning`), the agent **MUST read this registry** and only open the specific documentation files relevant to the feature being built.

---

## 1. Technical Services

*   [API Service](file:///d:/_skalfa/agent-api/knowledges/technical/api.md): Routing patterns, controller delegation, and model representation.
*   [Middleware](file:///d:/_skalfa/agent-api/knowledges/technical/middleware.md): Global vs. local (route group) middleware usage, and execution order.
*   [Database & Transactions](file:///d:/_skalfa/agent-api/knowledges/technical/database.md): ORM queries, record storage, raw Knex usage, and transaction management (`db.transaction`).
*   [File Storage](file:///d:/_skalfa/agent-api/knowledges/technical/storage.md): Public vs. private folders, private file access protection, and `uploadFile` & `deleteFile` helpers.
*   [Email Service](file:///d:/_skalfa/agent-api/knowledges/technical/email.md): SMTP configuration, `sendMail` function, and rendering layout/email templates.
*   [Cron Jobs](file:///d:/_skalfa/agent-api/knowledges/technical/cron.md): Registering tasks via `cron.add` and separate runtime rules for development vs. production.
*   [Queue Workers](file:///d:/_skalfa/agent-api/knowledges/technical/queue.md): Adding jobs via `queue.add` and writing asynchronous processors via `queue.worker`.
*   [WebSockets](file:///d:/_skalfa/agent-api/knowledges/technical/socket.md): Socket.IO server setup via `socket.start`, event registration with/without auth, emitting (`emit`, `send`, `room`), and queue integration.
*   [Data Analytics (OLAP Clickhouse)](file:///d:/_skalfa/agent-api/knowledges/technical/da.md): Purpose of separating OLTP and OLAP, Clickhouse queries (`da.query`), and batch inserting analytics data (`da.insert`).

---

## 2. Core Utilities

*   [ORM & Model](file:///d:/_skalfa/agent-api/knowledges/utilities/orm.md): Model definition, `@Field`, `@SoftDelete`, `@Attribute` decorators, relations, model hooks, relation filtering (`whereHas`), and filter operators.
*   [Controller Utility](file:///d:/_skalfa/agent-api/knowledges/utilities/controller.md): Extracting query parameters with `c.getQuery`, validating payloads with `c.validation`, and sending standard responses (`c.responseData`, `c.responseSaved`, etc.).
*   [Validation Rules](file:///d:/_skalfa/agent-api/knowledges/utilities/validation.md): How to trigger validation and the list of supported validation rules in Skalfa.
*   [Permissions (RBAC)](file:///d:/_skalfa/agent-api/knowledges/utilities/permission.md): Registering module permissions and securing controller methods via `guard`.
*   [Authentication & Session (Auth)](file:///d:/_skalfa/agent-api/knowledges/utilities/auth.md): JWT token generation/verification, temporary email verification tokens, and permission cache revalidation.
*   [Caching (Redis)](file:///d:/_skalfa/agent-api/knowledges/utilities/caching.md): Cache key generation, `cache.get`/`set`/`clear` operations, and cache-aside patterns.
*   [Converting](file:///d:/_skalfa/agent-api/knowledges/utilities/converting.md): Formatting string cases, Rupiah currency, and localized dates (`id-ID`).
*   [Encrypting](file:///d:/_skalfa/agent-api/knowledges/utilities/encrypting.md): Two-way encryption (AES, TripleDES) and one-way hashing (SHA256, SHA512, MD5).
*   [Migration Helpers](file:///d:/_skalfa/agent-api/knowledges/utilities/migration.md): Using `table.foreignIdFor` and `table.softDelete` in database migration files.
