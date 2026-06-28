# Panduan Teknis: Antrean Tugas (Queue Worker)

Queue di Skalfa API digunakan untuk menangani proses asynchronous dan background task (seperti pengiriman email, push notification, manipulasi berkas berat, atau integrasi pihak ketiga) yang tidak harus selesai di dalam siklus request HTTP utama.

---

## 1. Menambahkan Job ke Antrean (`queue.add`)

Untuk memasukkan pekerjaan ke dalam antrean, gunakan `queue.add`. Metode ini menerima nama antrean (queue name) dan objek payload data yang akan dikirim ke worker.

```typescript
import { queue } from '@utils'

// Menambahkan data ke antrean 'send-welcome-email'
await queue.add('send-welcome-email', {
  user_id: 12,
  email:   'joko@email.com',
  name:    'Joko Gunawan'
})
```

---

## 2. Menulis Queue Worker (`queue.worker`)

Pekerjaan yang masuk ke dalam antrean akan diambil dan diproses secara terpisah oleh fungsi worker yang telah terdaftar.

```typescript
// src/jobs/queues/workers.ts
import { queue, sendMail, renderMailTemplate } from '@utils'

// Mendaftarkan pemroses untuk antrean 'send-welcome-email'
queue.worker('send-welcome-email', async (payload, id) => {
  const html = renderMailTemplate('welcome', { name: payload.name });
  
  await sendMail({
    to:      payload.email,
    subject: 'Selamat Datang di Skalfa!',
    content: html
  });
  
  console.log(`Queue ${id} sukses mengirim email ke ${payload.email}`);
});
```

---

## 3. Menjalankan Queue Worker di Production

Sama seperti cron, di environment production **queue worker wajib dijalankan sebagai proses terpisah** dari server HTTP utama agar antrean panjang atau kegagalan worker tidak memengaruhi performa respon API pengguna.

```bash
# Menjalankan worker antrean secara mandiri di production
bun start:queue
```

---

## 4. Praktek Terbaik (Best Practice)
*   **Payload Sederhana**: Jaga agar payload data tetap kecil. Jangan mengirim objek database ORM yang besar atau fungsi callback. Kirimkan saja tipe data primitif seperti ID database (misal: `user_id: 12`), lalu biarkan worker memuat data terbaru dari database jika diperlukan.
*   **Idempoten**: Pastikan logika worker bersifat idempoten (aman jika dijalankan lebih dari sekali untuk payload yang sama jika terjadi proses coba-ulang/retry otomatis).
