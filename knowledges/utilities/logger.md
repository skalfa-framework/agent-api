# Panduan Utilitas: Pencatatan Log (Logger) (`@utils`)

Utilitas `logger` digunakan untuk mencetak pesan log berwarna di konsol terminal serta menulis log kesalahan secara otomatis ke dalam berkas fisik di server Skalfa API.

## 1. Pencatatan Konsol Berwarna

Pesan di konsol terminal akan diawali dengan label kategori yang berwarna-warni sesuai tipenya untuk mempermudah monitoring:

*   **`logger.start(msg)`**: Label hijau `[START]`, digunakan saat server pertama kali dinyalakan.
*   **`logger.info(msg)`**: Label sian `[INFO]`, untuk informasi aktivitas umum.
*   **`logger.warning(msg)`**: Label kuning `[WARNING]`, untuk pesan peringatan sistem.
*   **`logger.queue(msg)`**: Label biru `[QUEUE]`, untuk monitoring antrean tugas background.
*   **`logger.cron(msg)`**: Label magenta `[CRON]`, untuk eksekusi tugas terjadwal.
*   **`logger.socket(msg)`**: Label biru `[SOCKET]`, untuk aktivitas koneksi WebSocket.

---

## 2. Pencatatan Error ke Berkas (`logger.error`)

Mencatat kesalahan ke konsol dengan warna merah `[ERROR]` dan otomatis menulis log tersebut ke berkas fisik `storage/logs/error.log` dalam format JSON Line (JSONL) untuk audit kesalahan produksi.

```typescript
import { logger } from "@utils";

try {
  // ... operasi bermasalah ...
} catch (err) {
  const error = err as Error;
  
  // Mencatat ke konsol dan menulis ke storage/logs/error.log
  logger.error(error.message, {
    feature:   "BookingController.store", // Lokasi fitur yang error
    reference: "Booking ID #1092"         // Referensi data terkait
  });
}
```

---

## 3. Contoh Tampilan Berkas `error.log`
```json
{"at":"2026-06-28T21:58:00.000Z","error":"Database connection timeout","feature":"BookingController.store","reference":"Booking ID #1092"}
```
*Catatan untuk Agen: Selalu tangkap error menggunakan try-catch di controller dan catat kesalahannya menggunakan `logger.error` sebelum mengirimkan respon 500 ke klien.*
