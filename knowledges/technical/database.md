# Panduan Teknis: Database & Transaksi

Skalfa API menyediakan ORM ringan (`skalfa-orm`) yang dibangun di atas Knex.js. ORM ini tidak menggantikan Knex, melainkan memperluasnya dengan struktur model, helper query, dan integrasi langsung dengan `ControllerContext`. Developer tetap memiliki kendali penuh dan dapat menggunakan seluruh API Knex.js kapan pun dibutuhkan.

---

## 1. Query Dasar dengan ORM

Metode `query()` memulai query berbasis model. Metode `resolve(c)` secara otomatis mengikat parameter query dari `ControllerContext` (halaman, limit, pencarian, pengurutan, filter).

```typescript
import { User } from '@models'

// Mengambil data terfilter & terpaginasi otomatis
const users = await User.query().resolve(c)

c.responseData(users.data, users.total)
```

---

## 2. Membuat dan Menyimpan Record

ORM mendukung pendekatan instance-based maupun static helper:

```typescript
// Pendekatan Instance
const record = new User()
record.name = "Joko Gunawan"
await record.save()

// Pendekatan Static Helper
const record = await User.create({
  name: "Joko Gunawan"
})
```

---

## 3. Menggunakan Knex Secara Langsung

Untuk query kompleks, join tabel yang berat, agregasi database, atau operasi khusus, Anda dapat menggunakan Knex secara langsung melalui `db.query` (tanpa melewati model).

```typescript
import { db } from '@utils'

const activeUsers = await db.query
  .select('users.*', 'roles.name as role_name')
  .from('users')
  .join('roles', 'roles.id', 'users.role_id')
  .where('users.status', 'active')
  .orderBy('users.created_at', 'desc')
```

---

## 4. Transaksi Database (Transactions)

Untuk memastikan konsistensi data saat melakukan beberapa operasi tulis berturut-turut, wajib menggunakan transaksi (`db.transaction()`) yang dibungkus dengan blok try-catch.

```typescript
import { db } from '@utils'
import { Booking } from '@models'

const trx = await db.transaction()

try {
  // Simpan data utama beserta relasinya di dalam transaksi
  const record = await new Booking().pump(c.body, { trx })
  
  // Lakukan operasi lain jika diperlukan...
  
  await trx.commit()
  c.responseSaved(record)
} catch (err) {
  await trx.rollback()
  c.responseError(err as Error, 'Create Booking')
}
```

---

## 5. Migrasi & Seeder

*   **Database Migration**: Mengelola perubahan skema database secara versioned. Gunakan pembantu `foreignIdFor` dan `softDelete` untuk standarisasi pembuatan kolom.
*   **Database Seeder**: Berfungsi untuk mengisi data awal (master data seperti role, permission) maupun data tiruan untuk simulasi pengembangan.
*   **OLAP Migration**: Digunakan khusus untuk skema Clickhouse (analitik) yang terpisah dari database transaksional utama.
