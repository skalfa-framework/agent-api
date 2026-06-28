# Panduan Teknis: Penyimpanan Berkas (Storage)

Skalfa API menyediakan utilitas storage untuk menangani unggah (*upload*), penyimpanan, perlindungan hak akses, dan penghapusan file secara terstruktur.

---

## 1. Mengaktifkan Storage

Storage diaktifkan dengan memasang plugin/middleware `storage` pada instance Elysia. Ini akan mendaftarkan rute `/storage/*` untuk melayani berkas fisik secara otomatis.

```typescript
// src/app.ts
import { Elysia } from 'elysia'
import { storage } from '@utils'

export const app = new Elysia().use(storage)
```

---

## 2. Struktur Folder Storage

Berkas disimpan secara fisik di folder `storage` pada root project, yang terbagi menjadi dua jenis penyimpanan:

```text
storage/
├─ public/               # Akses bebas tanpa autentikasi (gambar profil, dsb)
│  └─ avatars/
└─ private/              # Membutuhkan otorisasi & pengecekan izin khusus
   └─ invoices/
```

---

## 3. Akses Berkas (Public vs Private)

*   **Public Storage**: File diakses langsung lewat URL:
    `http://localhost:4000/storage/avatars/image.png`
*   **Private Storage**: Akses ke `/storage/private/*` akan ditangkap oleh middleware. Middleware akan memeriksa:
    1.  Apakah pengguna sedang login (`user` terdefinisi).
    2.  Apakah pengguna adalah pemilik berkas tersebut (`user_id` cocok di tabel `storages`).
    3.  Apakah pengguna atau role pengguna memiliki izin khusus yang tercatat di tabel `storage_permissions`.
    *   *Hasil*: Jika tidak memiliki hak akses, server mengembalikan respon `404 File not found` (menyembunyikan keberadaan file demi privasi).

---

## 4. Mengunggah Berkas (`uploadFile`)

Unggah dilakukan di controller menggunakan `uploadFile` (atau disuntikkan ke `c.uploadFile`). Metode ini menyimpan file fisik dan mencatat metadatanya ke database.

```typescript
import { uploadFile } from "@utils";

export class FileController {
  static async upload(c: ControllerContext) {
    const file = c.body.file as File;

    // Unggah sebagai berkas privat
    const path = await uploadFile(file, 'invoices', {
      disk:        'private',
      owner_id:    c.user.id,
      permissions: [
        { role_id: 1 } // Izinkan Admin melihat invoice ini
      ]
    });

    c.responseSuccess({ path })
  }
}
```

---

## 5. Menghapus Berkas (`deleteFile`)

Menghapus file dari disk fisik dan otomatis membersihkan catatan metadata serta izinnya di database.

```typescript
import { deleteFile } from "@utils";

// Hapus file fisik & record DB
await deleteFile('/invoices/abc123xyz.pdf');
```
