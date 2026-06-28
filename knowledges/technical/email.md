# Panduan Teknis: Layanan Email (Email)

Skalfa API menyediakan layanan email berbasis SMTP untuk memicu pengiriman pesan keluar secara transaksional dengan dukungan perenderan template email dinamis.

---

## 1. Konfigurasi Server SMTP

Koneksi SMTP dikonfigurasi melalui environment variables di dalam berkas `.env`:
```text
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=my_username
MAIL_PASSWORD=my_password
MAIL_FROM_ADDRESS=noreply@my-app.com
MAIL_FROM_NAME="Skalfa App"
```

---

## 2. Mengirim Email (`sendMail`)

Gunakan fungsi `sendMail` untuk mengirim email. Fungsi ini berjalan secara asinkron dan menerima opsi penerima, subjek, isi pesan (teks/HTML), serta lampiran berkas.

```typescript
import { sendMail } from "@utils";

await sendMail({
  to:          "user@target.com",
  subject:     "Tagihan Pembayaran #1029",
  content:     "<h1>Halo!</h1><p>Silakan temukan tagihan Anda terlampir.</p>", // HTML
  text:        "Halo! Silakan temukan tagihan Anda terlampir.",                 // Teks biasa
  attachments: [
    {
      filename: "invoice-1029.pdf",
      path:     "storage/private/invoices/invoice-1029.pdf"
    }
  ]
});
```

---

## 3. Rendering Template Email (`renderMailTemplate`)

Agar desain email tetap konsisten, pisahkan tata letak (*layout*) dan konten email menggunakan template stub di folder `outputs/mails/templates/`.

*   **Layout Utama** (`layout.mail.stub`): Menyediakan struktur HTML standar, header, dan footer global.
*   **Template Konten** (contoh: `reset-password.mail.stub`): Berisi pesan spesifik dengan variabel pembungkus `{{ nama_variabel }}`.

### Contoh Implementasi:
```typescript
import { sendMail, renderMailTemplate } from "@utils";

// Merender template dengan mengganti placeholder secara dinamis
const htmlBody = renderMailTemplate("reset-password", {
  username:  "Joko Gunawan",
  reset_url: "https://my-app.com/reset?token=xyz"
});

await sendMail({
  to:      "joko@email.com",
  subject: "Atur Ulang Kata Sandi Anda",
  content: htmlBody
});
```
*Catatan untuk Agen: Pindahkan selalu proses pengiriman email ke dalam Queue Worker (background job) agar tidak memperlambat respon API saat pengguna memicu aksi yang mengirimkan email.*
