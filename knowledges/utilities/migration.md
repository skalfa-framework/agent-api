# Panduan Utilitas: Pembantu Migrasi Database (Migration) (`@utils`)

Skalfa ORM memperluas antarmuka pembangun tabel Knex (`Knex.CreateTableBuilder`) dengan menambahkan fungsi pembantu untuk standarisasi pembuatan kunci asing (*foreign keys*) dan penanganan kolom soft delete di berkas migrasi database.

## 1. Kunci Asing Otomatis (`foreignIdFor`)

Metode `table.foreignIdFor(tableName, column?)` menyederhanakan pembuatan kolom referensi kunci asing.
*   **Tipe Data**: Otomatis membuat kolom bertipe `unsignedBigInteger`.
*   **Indeks**: Otomatis menambahkan indeks ke kolom tersebut.
*   **Relasi**: Otomatis membuat relasi rujukan ke tabel target (`references("id").on(tableName)`) dengan aksi penghapusan berskala (`onDelete("CASCADE")`).

### Contoh Penggunaan:
```typescript
import { Knex } from "knex";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable("bookings", (table) => {
    table.increments("id").primary();
    
    // Otomatis membuat kolom 'user_id' yang merujuk ke tabel 'users(id)'
    table.foreignIdFor("users"); 
    
    // Atau tentukan nama kolom kustom (jika berbeda dari nama_tabel_id)
    table.foreignIdFor("users", "customer_id");
  });
}
```

---

## 2. Penanda Hapus Logis (`softDelete`)

Metode `table.softDelete(column?)` membuat kolom penanda penghapusan logis secara standar.
*   **Tipe Data**: Membuat kolom bertipe `timestamp` nullable.
*   **Nama Kolom**: Default `"deleted_at"`.

### Contoh Penggunaan:
```typescript
export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable("bookings", (table) => {
    table.increments("id").primary();
    // ... kolom lainnya ...

    table.softDelete(); // Hasil: Membuat kolom 'deleted_at'
  });
}
```
*Catatan untuk Agen: Selalu gunakan pembantu `foreignIdFor` dan `softDelete` saat menulis berkas migrasi tabel baru guna menjaga konsistensi skema relasi database.*
