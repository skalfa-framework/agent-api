# Utility Guide: ORM & Model (`orm`)

Models in Skalfa API are built on top of Knex.js, providing an Active Record ORM. They handle schema definition, relations, casting, lifecycle hooks, and query builders.

---

## 1. Defining a Model

Models inherit from the abstract `Model` class. Columns are defined using the `@Field` decorator.

```typescript
import { Model, Field, SoftDelete, Attribute, BelongsToMany } from '@utils'
import { Role } from '@models'

export class User extends Model {
  static table = "users";

  @Field(['string', 'fillable', 'selectable', 'searchable'])
  name!: string

  @Field(['string', 'fillable', 'selectable'])
  email!: string

  // Computed Attribute
  @Attribute()
  username() {
    if (!this.name) return null;
    return this.name.toLowerCase().replace(/\s+/g, '');
  }

  @SoftDelete()
  deleted_at!: Date

  @BelongsToMany(() => Role, 'user_roles', 'user_id', 'role_id')
  roles!: Role[]
}
```

### `@Field` Options:
*   `string` / `number` / `boolean` / `date` / `json`: Automatic data type casting.
*   `fillable`: Field is allowed to be populated via `.pump()`.
*   `selectable`: Field is included in default SELECT queries.
*   `searchable`: Field is scanned during global `.search()` queries.

---

## 2. Querying Data

Querying starts with `Model.query()`, which returns a Knex query builder extended with several helpers:

### A. Searching (`.search`)
```typescript
const users = await User.query()
  .search(c.getQuery.search, {
    searchables: ['name'], // Optional override
    includes:    ['email'] // Optional additions
  })
  .get()
```

### B. Filtering (`.filter`)
Filters records using a standardized `operator:value` syntax.
*   *Operators*: `li` (ilike), `eq` (=), `ne` (!=), `in` (whereIn), `ni` (whereNotIn), `bw` (between).
```typescript
// URL format: ?filter={"name":"li:john"}
await User.query().filter(c.getQuery.filter).get()
```

### C. Eager Loading Relations (`.expand`)
Loads related models automatically.
```typescript
await User.query().expand(["roles"]).get()
```

### D. Pagination & Option Mode (`.paginateOrOption`)
Resolves queries into either a paginated list or a simple `{ value, label }` dropdown list based on the presence of `isOption=true` query or `x-options` header.
```typescript
await User.query().paginateOrOption(
  c.getQuery.page,
  c.getQuery.paginate,
  c.request.headers.get("x-options") === "true",
  ["id", "name"] // [value, label] mapping
)
```

---

## 3. Model Relations

*   `@HasOne(() => TargetModel, foreignKey)`
*   `@HasMany(() => TargetModel, foreignKey)`
*   `@BelongsTo(() => TargetModel, foreignKey)`
*   `@BelongsToMany(() => TargetModel, pivotTable, foreignKey, targetKey)`

---

## 4. Lifecycle Hooks (`@On`)

Hooks intercept database operations before or after saving. Define them as instance methods decorated with `@On(event)`.

### A. Supported Events
*   `before-create` / `after-create`
*   `before-update` / `after-update`
*   `before-delete` / `after-delete`

### B. Example
```typescript
import { Model, On } from '@utils'

export class User extends Model {
  // ...

  @On('before-create')
  beforeCreate({ trx }) {
    if (this.name) {
      this.name = this.name.trim();
    }
  }
}
```

### C. Modifying Hooks from Controllers
```typescript
const user = new User();
user.off('before-create'); // Turn off hook
```
---

## 5. Relation Filters (`whereHas` & `whereDoesntHave`)

Filters parent records based on conditions in their related tables.

```typescript
const users = await User.query().whereHas('posts', (q) => {
  q.where('posts.title', 'like', '%announcement%');
});
```
