# Registri Pengetahuan Skalfa (Knowledge Registry - Backend)

Sebelum membuat rencana implementasi (`Stage 1 — Planning`), agen **wajib membaca indeks ini** dan hanya membuka berkas dokumentasi spesifik yang relevan dengan fitur yang akan dibuat.

---

## 1. Fitur Teknis (Technical Services)

*   [Layanan API (API Service)](file:///d:/_skalfa/agent-backend/knowledges/technical/api.md): Pola routing group, delegasi ke controller, dan representasi model.
*   [Pemasangan Middleware](file:///d:/_skalfa/agent-backend/knowledges/technical/middleware.md): Penggunaan middleware global vs lokal (grup rute), serta urutan eksekusinya.
*   [Database & Transaksi](file:///d:/_skalfa/agent-backend/knowledges/technical/database.md): ORM query, penyimpanan record, penggunaan raw Knex, dan pengelolaan transaksi database (`db.transaction`).
*   [Penyimpanan Berkas (Storage)](file:///d:/_skalfa/agent-backend/knowledges/technical/storage.md): Folder public vs private, perlindungan hak akses berkas privat, serta fungsi `uploadFile` & `deleteFile`.
*   [Layanan Email (Email)](file:///d:/_skalfa/agent-backend/knowledges/technical/email.md): Konfigurasi SMTP, fungsi `sendMail`, dan rendering layout/template email stub.
*   [Tugas Terjadwal (Cron Job)](file:///d:/_skalfa/agent-backend/knowledges/technical/cron.md): Pendaftaran `cron.add` dan aturan runtime terpisah di development vs production.
*   [Antrean Tugas (Queue Worker)](file:///d:/_skalfa/agent-backend/knowledges/technical/queue.md): Penambahan job via `queue.add` dan penulisan pemroses asinkron via `queue.worker`.
*   [WebSocket Realtime (Socket)](file:///d:/_skalfa/agent-backend/knowledges/technical/socket.md): Socket.IO server `socket.start`, registrasi event dengan/tanpa auth, pemancaran (`emit`, `send`, `room`), dan integrasi dengan queue.
*   [Analitik Data (OLAP Clickhouse)](file:///d:/_skalfa/agent-backend/knowledges/technical/da.md): Mengapa memisahkan OLTP dan OLAP, query Clickhouse (`da.query`), dan pengisian batch data analitik (`da.insert`).

---

## 2. Utilitas Inti (Core Utilities)

*   [ORM & Model](file:///d:/_skalfa/agent-backend/knowledges/utilities/orm.md): Pendefinisian model, decorator `@Field`, `@SoftDelete`, `@Attribute`, relasi, hook model, penyaringan relasi (`whereHas`), dan filter operator.
*   [Pengendali (Controller Utility)](file:///d:/_skalfa/agent-backend/knowledges/utilities/controller.md): Pengambilan query `c.getQuery`, validasi payload `c.validation`, pengiriman respons standar (`c.responseData`, `c.responseSaved`, dsb).
*   [Aturan Validasi (Validation)](file:///d:/_skalfa/agent-backend/knowledges/utilities/validation.md): Cara memicu validasi dan daftar lengkap aturan validasi yang didukung Skalfa.
*   [Hak Akses & Pengaman (Permission)](file:///d:/_skalfa/agent-backend/knowledges/utilities/permission.md): Pendaftaran izin modul dan pengamanan metode controller via `guard`.
*   [Autentikasi Sesi (Auth)](file:///d:/_skalfa/agent-backend/knowledges/utilities/auth.md): Pembuatan/verifikasi JWT token, token email sekali pakai, serta revalidasi cache permission.
*   [Caching (Redis)](file:///d:/_skalfa/agent-backend/knowledges/utilities/caching.md): Pembuatan key cache, operasi `cache.get`/`set`/`clear`, dan pola cache-aside.
*   [Konversi Format (Converting)](file:///d:/_skalfa/agent-backend/knowledges/utilities/converting.md): Fungsi pemformatan huruf string, nominal Rupiah, dan tanggal lokalisasi `id-ID`.
*   [Enkripsi Data (Encrypting)](file:///d:/_skalfa/agent-backend/knowledges/utilities/encrypting.md): Enkripsi/dekripsi dua arah (AES, TripleDES) dan hash satu arah (SHA256, SHA512, MD5).
*   [Pembantu Migrasi (Migration)](file:///d:/_skalfa/agent-backend/knowledges/utilities/migration.md): Penggunaan `table.foreignIdFor` dan `table.softDelete` pada berkas migrasi database.
