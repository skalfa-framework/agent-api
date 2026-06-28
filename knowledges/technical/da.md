# Panduan Teknis: Analitik Data (OLAP Clickhouse)

Skalfa API menyediakan utilitas analitik data berbasis ClickHouse untuk kebutuhan Online Analytical Processing (OLAP) seperti penyimpanan access logs, error logs, audit trail, event tracking, dan pelaporan statistik skala besar.

---

## 1. Mengapa Memisahkan OLTP dan OLAP?

*   **OLTP (PostgreSQL/MySQL)**: Digunakan oleh ORM/Knex untuk transaksi harian aplikasi (tabel bookings, users, payments). Dirancang untuk konsistensi tinggi dan query baris tunggal yang cepat.
*   **OLAP (ClickHouse)**: Digunakan khusus untuk analitik dan logging data besar. Dirancang untuk query agregasi kolom yang sangat cepat pada miliaran baris data tanpa membebani database operasional utama (OLTP).

---

## 2. Membaca Data Analitik (`da.query`)

Gunakan `da.query()` untuk mengambil data dari ClickHouse menggunakan query builder terabstraksi.

```typescript
import { da } from "@utils";

const logs = await da.query()
  .from('error_logs')
  .where('level', 'ERROR')
  .andWhere('created_at', '>', '2026-06-01')
  .limit(100)
  .get();
```

---

## 3. Memasukkan Data (`da.insert`)

ClickHouse dirancang sangat efisien untuk proses memasukkan data dalam jumlah banyak sekaligus (*batch insert*). Hindari melakukan satu per satu insert (single insert) secara berulang.

```typescript
import { da } from "@utils";

await da.insert({
  table:  'error_logs',
  format: 'JSONEachRow', // Format transfer data ClickHouse
  values: [
    { message: "Database timeout", level: "ERROR", created_at: new Date() },
    { message: "Page not found",    level: "WARNING", created_at: new Date() }
  ]
});
```

---

## 4. Praktek Terbaik (Best Practice)
*   **Batching**: Selalu kumpulkan data analitik terlebih dahulu (misalnya di antrean memory atau queue) lalu lakukan insert secara massal (*batch*) setiap beberapa detik atau setelah mencapai jumlah baris tertentu.
*   **Append-Only**: Data ClickHouse dirancang bersifat *append-only* (tambah saja). Hindari merancang skema analitik yang membutuhkan operasi `UPDATE` atau `DELETE` baris data secara berkala, karena operasi tersebut sangat mahal di ClickHouse.
