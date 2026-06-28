# Panduan Teknis: Middleware

Middleware di Skalfa API digunakan untuk menangani proses lintas endpoint seperti CORS, parsing request body, autentikasi, logging, dan validasi. Skalfa API mengikuti cara kerja native Elysia, tanpa menambahkan abstraksi tersembunyi, sehingga alur eksekusi middleware tetap eksplisit dan mudah dipahami.

---

## 1. Konsep Dasar Middleware

Middleware adalah fungsi yang dieksekusi sebelum handler endpoint dijalankan. Ia dapat:
1.  Memodifikasi request (misalnya: mendekripsi token).
2.  Menambahkan data ke konteks (misalnya: menyuntikkan data `c.user`).
3.  Menghentikan request (misalnya: jika otentikasi gagal).
4.  Melanjutkan proses ke handler berikutnya.

---

## 2. Pemasangan Middleware

### A. Middleware Global (Chaining di Instance)
Diberlakukan untuk seluruh request yang masuk ke aplikasi. Biasanya dipasang saat inisialisasi server.

```typescript
// src/app.ts
import { Elysia } from 'elysia'
import { Middleware } from '@utils'

const app = new Elysia()
  .use(Middleware.Cors)
  .use(Middleware.BodyParse)
```

### B. Middleware Lokal (Di dalam Group / Route)
Hanya berlaku untuk endpoint yang didefinisikan setelah middleware tersebut dipanggil di dalam grup rute.

```typescript
// src/routes/index.ts
export const routes = (app: Elysia) => app.group('/api', (route) => {
  // Endpoint ini bebas akses (tidak melewati Middleware.Auth)
  route.post('/login', AuthController.login)

  // Mengunci semua endpoint di bawah baris ini dengan Middleware.Auth
  route.use(Middleware.Auth)

  // Endpoint ini wajib terotentikasi
  route.get('/me', AuthController.me)
  route.get('/users', UserController.index)

  return route
})
```

---

## 3. Urutan Eksekusi

Urutan penulisan middleware menentukan alur eksekusinya:
1.  **Middleware Global** → Dijalankan pertama kali pada setiap request.
2.  **Middleware Group / Route** → Dijalankan sesuai urutan deklarasi baris kode di dalam route group.
3.  **Controller / Handler** → Dieksekusi terakhir setelah lolos seluruh middleware.

---

## 4. Kapan Menggunakan Global vs Lokal

*   **Global**: Gunakan untuk penanganan teknis umum seperti CORS, request logger, body parser, dan global error handler.
*   **Lokal / Group**: Gunakan untuk aturan bisnis spesifik seperti pengecekan login (`Auth`), verifikasi peran (`Role`), validasi izin (`Permission`), atau validasi khusus rute tertentu.
