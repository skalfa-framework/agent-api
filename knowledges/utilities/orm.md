# Panduan Utilitas: ORM & Model (`@utils`)

Model di Skalfa API adalah lapisan ORM ringan yang dibangun di atas Knex.js. Model bertugas mendefinisikan skema data, relasi, casting, hook lifecycle, serta helper query yang terintegrasi langsung dengan `ControllerContext`.

---

## 1. Mendefinisikan Model

Model dibuat dengan mewarisi kelas abstrak `Model`. Properti kolom didefinisikan menggunakan dekorator `@Field`.

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

### Kunci Parameter `@Field`:
*   `string` / `number` / `boolean` / `date` / `json`: Casting tipe data otomatis dari/ke database.
*   `fillable`: Kolom diizinkan disimpan melalui payload `.pump()`.
*   `selectable`: Kolom dimasukkan ke dalam daftar select default.
*   `searchable`: Kolom diikutsertakan dalam pencarian teks.

---

## 2. Query Menggunakan Model

Query diawali dengan `Model.query()`, yang menghasilkan query builder Knex yang telah diperluas dengan berbagai helper:

### A. Pencarian (`.search`)
```typescript
const users = await User.query()
  .search(c.getQuery.search, {
    searchables: ['name'], // Opsional: menimpa kolom searchable model
    includes:    ['email'] // Opsional: menambahkan kolom pencarian
  })
  .get()
```

### B. Penyaringan (`.filter`)
Mendukung operator penyaringan terstandardisasi menggunakan format `operator:nilai`.
*   *Operator*: `li` (ilike), `eq` (=), `ne` (!=), `in` (whereIn), `ni` (whereNotIn), `bw` (between).
*   *Kondisi*: `or` atau `and`.

```typescript
// Format string: "field=operator:nilai"
await User.query().filter(c.getQuery.filter).get()

// Secara programmatik:
await User.query().filter({
  name:    'li:joko',   // name ILIKE '%joko%'
  role_id: 'eq:1',      // role_id = 1
  status:  'or:eq:active' // OR status = 'active'
}).get()
```

### C. Relasi Eager Loading (`.expand`)
Memuat relasi model secara otomatis menggunakan pemisah titik dua (`:`) untuk kolom spesifik.

```typescript
// Memuat relasi 'roles' hanya untuk kolom 'id' dan 'name'
await User.query().expand(["roles:id,name"]).get()
```

### D. Paginasi & Mode Opsi (`.paginateOrOption`)
Mendukung pencarian list biasa (paginasi) atau format dropdown pilihan (options mode) secara dinamis berbasis header `x-options` atau query `isOption=true`.

```typescript
// Jika mode options aktif, mengembalikan array: { value: id, label: name }
await User.query().paginateOrOption(
  c.getQuery.page,
  c.getQuery.paginate,
  c.request.headers.get("x-options") === "true",
  ["id", "name"] // Kolom [value, label] untuk mode option
)
```

---

## 3. Relasi Antar Model

*   **`@HasOne(() => TargetModel, foreignKey)`**
*   **`@HasMany(() => TargetModel, foreignKey)`**
*   **`@BelongsTo(() => TargetModel, foreignKey)`**
*   **`@BelongsToMany(() => TargetModel, pivotTable, foreignKey, targetKey)`**

---

## 4. Lifecycle Hooks Model

Model mendukung hook untuk memotong alur kerja database sebelum/sesudah data disimpan, diperbarui, atau dihapus.

### A. Jenis Hook
*   `before-create` / `after-create`
*   `before-update` / `after-update`
*   `before-delete` / `after-delete`

### B. Pendaftaran Hook Menggunakan Decorator (`@On`)
Hook didefinisikan sebagai *instance method* di dalam kelas model dan dihias dengan dekorator `@On(event)`. Di dalam method ini, Anda dapat merujuk ke instance model aktif menggunakan `this`.

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

  @On('after-create')
  afterCreate({ trx }) {
    // Aksi setelah berhasil disimpan...
  }
}
```

### C. Modifikasi Hook dari Controller (Instance-based)
*   **Mematikan Hook (`model.off`)**:
    ```typescript
    const user = new User();
    user.off('before-create'); // Menonaktifkan hook before-create saat user.save()
    ```
*   **Mendaftarkan Hook Dinamis (`model.on`)**:
    ```typescript
    const user = new User();
    user.on('before-create', ({ model, trx }) => {
      model.name = model.name.toUpperCase();
    });
    ```
---

## 5. Filter Relasi (`whereHas` & `whereDoesntHave`)

Menyaring data model berdasarkan kondisi pada tabel relasinya.

```typescript
// Hanya mengambil user yang memiliki postingan dengan judul mengandung 'test'
const users = await User.query().whereHas('posts', (q) => {
  q.where('posts.title', 'like', '%test%');
});
```
