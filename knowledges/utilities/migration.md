# Utility Guide: Migration Helpers (`migration`)

The `migration` utility provides custom extension methods to Knex's Schema Builder, simplifying common table definitions.

---

## 1. Custom Table Helpers

These helpers are available on the Knex `TableBuilder` instance in your migration files.

### A. Soft Delete Column (`table.softDelete()`)
Adds a nullable `deleted_at` timestamp column to support soft deleting records.
```typescript
// database/migrations/2026_06_create_users_table.ts
import { Knex } from "knex";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable("users", (table) => {
    table.increments("id").primary();
    table.string("name");
    
    // Adds deleted_at timestamp
    table.softDelete();
  });
}
```

### B. Foreign ID Column (`table.foreignIdFor(targetTable, columnName?)`)
Adds an unsigned integer foreign key column and sets up the cascading foreign key constraint automatically.
```typescript
// database/migrations/2026_06_create_posts_table.ts
import { Knex } from "knex";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable("posts", (table) => {
    table.increments("id").primary();
    table.string("title");
    
    // Adds 'user_id' column referencing 'id' on 'users' table
    // with ON DELETE CASCADE and ON UPDATE CASCADE
    table.foreignIdFor("users");
    
    // Custom column name
    table.foreignIdFor("users", "author_id");
  });
}
```
---

## 2. Standard Migrations Workflow

1.  **Generate Migration**:
    ```bash
    skalfa make:migration create_users_table
    ```
2.  **Run Migrations**:
    ```bash
    skalfa migrate
    ```
3.  **Fresh Reset (Rollback & Re-run)**:
    ```bash
    skalfa migrate:fresh
    ```
