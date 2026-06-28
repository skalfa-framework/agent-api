# Panduan Teknis: Tugas Terjadwal (Cron Job)

Skalfa API menyediakan utilitas cron job untuk menjalankan tugas terjadwal seperti sinkronisasi data, pembersihan cache, pengiriman laporan periodik, atau rekapitulasi data secara otomatis tanpa bergantung pada request HTTP.

---

## 1. Mendaftarkan Cron Job (`cron.add`)

Cron job didefinisikan menggunakan utilitas `cron.add`. Setiap cron wajib memiliki pola jadwal (cron expression), fungsi handler, dan nama unik job untuk keperluan logging.

```typescript
// src/jobs/crons/workers.ts
import { cron } from '@utils'
import { BookingService } from '@services'

// Format: cron.add(scheduleExpression, handler, jobName)
cron.add('0 0 * * *', async () => {
  logger.info("Memulai pembersihan transaksi kedaluwarsa...");
  await BookingService.cleanupExpiredBookings();
}, 'cleanup-bookings')
```

### Penjelasan Parameter:
1.  **Schedule Expression**: Pola standardisasi 5-field cron (Menit, Jam, Hari dari Bulan, Bulan, Hari dari Minggu).
    *   `'0 0 * * *'` -> Berjalan setiap hari pada pukul 00:00 (tengah malam).
    *   `'*/5 * * * *'` -> Berjalan setiap 5 menit.
2.  **Handler**: Fungsi asinkron yang akan dieksekusi sesuai jadwal.
3.  **Job Name**: Nama unik yang dicatat ke `logger.cron` saat tugas dimulai, sukses, atau gagal.

---

## 2. Menjalankan Cron Job

### A. Environment Development
Pada mode development, cron job otomatis berjalan bersamaan di proses yang sama dengan server API utama saat Anda memicu perintah:
```bash
bun dev
```

### B. Environment Production
Di environment production, **cron job wajib dijalankan sebagai proses terisolasi** terpisah dari server HTTP utama agar tugas berat tidak memblokir server API.
```bash
# Menjalankan worker cron secara mandiri
bun start:cron
```

---

## 3. Praktek Terbaik (Best Practice)
*   **Pisahkan Logika**: Hindari menulis logika bisnis berat langsung di dalam handler cron. Buatlah method di dalam Service Class, lalu panggil method tersebut dari handler cron.
*   **Gunakan Logging**: Selalu catat log awal dan akhir eksekusi menggunakan `logger.info` atau `logger.cron` untuk memudahkan monitoring berkala.
