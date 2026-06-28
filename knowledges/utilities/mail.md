# Panduan Utilitas: Pengiriman Email (Mail) (`@utils`)

Utilitas `mail` digunakan untuk mengirim email berbasis SMTP dan merender template email menggunakan stub berkas di backend Skalfa API.

## 1. Mengirim Email (`sendMail`)

Metode `sendMail` menggunakan pustaka `nodemailer` di bawah kap dan secara otomatis membaca kredensial SMTP dari berkas `.env` (`MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_FROM_NAME`, `MAIL_FROM_ADDRESS`).

```typescript
import { sendMail } from "@utils";

await sendMail({
  to:          "penerima@email.com",
  subject:     "Verifikasi Pendaftaran Akun",
  content:     "<h1>Halo!</h1><p>Terima kasih telah mendaftar.</p>", // Konten HTML
  text:        "Halo! Terima kasih telah mendaftar.",              // Konten teks biasa (fallback)
  attachments: [
    { filename: "panduan.pdf", path: "/path/to/panduan.pdf" }
  ]
});
```

---

## 2. Merender Template Email (`renderMailTemplate`)

Merender template email menggunakan berkas stub di folder `outputs/mails/templates/<nama_template>.mail.stub` dan mengganti variabel placeholder `{{ key }}` secara otomatis, serta membungkusnya ke dalam layout utama `layout.mail.stub`.

### Contoh Berkas `verification.mail.stub`:
```html
<p>Halo {{ name }},</p>
<p>Silakan klik tautan berikut untuk memverifikasi akun Anda:</p>
<a href="{{ url }}">Verifikasi Akun</a>
```

### Pemanggilan di Controller / Service:
```typescript
import { sendMail, renderMailTemplate } from "@utils";

const htmlContent = renderMailTemplate("verification", {
  name: "Ahmad Budi",
  url:  "https://my-app.com/verify?token=xyz"
});

await sendMail({
  to:      "budi@email.com",
  subject: "Verifikasi Akun Anda",
  content: htmlContent
});
```
*Catatan untuk Agen: Selalu gunakan `renderMailTemplate` untuk menjaga keselarasan desain visual email keluar (seperti header/footer bermerek).*
