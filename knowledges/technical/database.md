# Technical Guide: Database & Transactions (`database`)

Skalfa API uses Knex.js as the query builder and database abstraction layer. Transactions and raw queries are handled via the `db` utility.

---

## 1. Basic Querying

For standard database operations, prefer using the Model-level query builder. However, for complex queries, join operations, or performance-critical tasks, you can use the raw `db` instance:

```typescript
import { db } from '@utils'

// Simple select
const activeUsers = await db("users")
  .where("status", "active")
  .select("id", "name", "email");

// Joins
const userRoles = await db("users")
  .join("user_roles", "users.id", "user_roles.user_id")
  .join("roles", "user_roles.role_id", "roles.id")
  .select("users.name", "roles.name as role_name");
```

---

## 2. Transactions

When executing multiple insert, update, or delete operations that depend on each other, you **MUST** wrap them in a database transaction to ensure data integrity (ACID).

```typescript
import { db } from '@utils'
import { User, Wallet } from '@models'

export class UserRegistrationService {
  static async register(payload: any) {
    // Start transaction
    const trx = await db.transaction();

    try {
      // 1. Create User
      const user = await new User().pump(payload, { trx });

      // 2. Create Wallet for User
      const wallet = await new Wallet().pump({
        user_id: user.id,
        balance: 0
      }, { trx });

      // Commit transaction if all succeeded
      await trx.commit();
      
      return { user, wallet };
    } catch (error) {
      // Rollback transaction if any operation failed
      await trx.rollback();
      throw error;
    }
  }
}
```

---

## 3. Raw Queries

Avoid raw SQL queries unless absolutely necessary. If required, use `db.raw`:

```typescript
const result = await db.raw("SELECT COUNT(*) as total FROM users WHERE created_at > ?", [someDate]);
```
