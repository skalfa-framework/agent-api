# Skalfa Knowledge Registry (Backend)

Before creating any implementation plan (`Stage 1 — Planning`), the agent **MUST read this registry** and only open the specific documentation files relevant to the feature being built.

---

## 1. Technical Services

*   [API Service](technical/api.md): Routing patterns, controller delegation, and model representation.
*   [Middleware](technical/middleware.md): Global vs. local (route group) middleware usage, and execution order.
*   [Database & Transactions](technical/database.md): ORM queries, record storage, raw Knex usage, and transaction management (`db.transaction`).
*   [File Storage](technical/storage.md): Public vs. private folders, private file access protection, and `uploadFile` & `deleteFile` helpers.
*   [Email Service](technical/email.md): SMTP configuration, `sendMail` function, and rendering layout/email templates.
*   [Cron Jobs](technical/cron.md): Registering tasks via `cron.add` and separate runtime rules for development vs. production.
*   [Queue Workers](technical/queue.md): Adding jobs via `queue.add` and writing asynchronous processors via `queue.worker`.
*   [WebSockets](technical/socket.md): Socket.IO server setup via `socket.start`, event registration with/without auth, emitting (`emit`, `send`, `room`), and queue integration.
*   [Data Analytics (OLAP Clickhouse)](technical/da.md): Purpose of separating OLTP and OLAP, Clickhouse queries (`da.query`), and batch inserting analytics data (`da.insert`).

---

## 2. Core Utilities

*   [CLI Blueprint](utilities/blueprint.md): Defining schemas, relations, seeders, and generating models/controllers/migrations.
*   [ORM & Model](utilities/orm.md): Model definition, `@Field`, `@SoftDelete`, `@Attribute` decorators, relations, model hooks, relation filtering (`whereHas`), and filter operators.
*   [Controller Utility](utilities/controller.md): Extracting query parameters with `c.getQuery`, validating payloads with `c.validation`, and sending standard responses (`c.responseData`, `c.responseSaved`, etc.).
*   [Validation Rules](utilities/validation.md): How to trigger validation and the list of supported validation rules in Skalfa.
*   [Permissions (RBAC)](utilities/permission.md): Registering module permissions and securing controller methods via `guard`.
*   [Authentication & Session (Auth)](utilities/auth.md): JWT token generation/verification, temporary email verification tokens, and permission cache revalidation.
*   [Caching (Redis)](utilities/caching.md): Cache key generation, `cache.get`/`set`/`clear` operations, and cache-aside patterns.
*   [Converting](utilities/converting.md): Formatting string cases, Rupiah currency, and localized dates (`id-ID`).
*   [Encrypting](utilities/encrypting.md): Two-way encryption (AES, TripleDES) and one-way hashing (SHA256, SHA512, MD5).
*   [Migration Helpers](utilities/migration.md): Using `table.foreignIdFor` and `table.softDelete` in database migration files.
