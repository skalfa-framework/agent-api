# Panduan Utilitas: Konteks Asinkron Global (Context) (`@utils`)

Utilitas `context` berbasis Node.js `AsyncLocalStorage` yang memungkinkan Anda menyimpan dan mengakses properti global (seperti ID pengguna yang sedang masuk) secara aman di seluruh alur eksekusi asinkron tanpa harus meneruskan variabel konteks Elysia `c` secara manual di setiap parameter fungsi.

## 1. Mengapa Menggunakan `context`?

Pada arsitektur backend, sering kali kita memerlukan informasi pengguna aktif (misalnya `user_id`) di tempat yang tidak memiliki akses ke objek request Elysia `c`, seperti:
*   **Model Hooks / Objek ORM** (seperti sebelum menyimpan data di hook `beforeSave`).
*   **Layanan Bisnis / Service Objects** di lapisan terdalam.
*   **Query Builder Helpers**.

Dengan `context`, data ini dapat dibaca dari mana saja selama masih berada dalam alur eksekusi asinkron yang sama.

---

## 2. Metode yang Tersedia

### A. Menjalankan Konteks (`context.run`)
Mendaftarkan scope asinkron beserta data konteksnya. Biasanya dieksekusi di tingkat middleware otentikasi.

```typescript
import { context } from "@utils";

// Format: context.run(dataContext, callback)
context.run({ user_id: activeUser.id }, () => {
  // Semua fungsi asinkron di dalam scope ini dapat mengakses user_id
  next();
});
```

### B. Mengambil Nilai Konteks (`context.get`)
Mengambil nilai properti dari konteks yang aktif.

```typescript
import { context } from "@utils";

const userId = context.get("user_id"); // Mengembalikan number atau undefined
```

---

## 3. Contoh Penggunaan di Model ORM

Digunakan untuk mencatat pembuat data (*blameable columns*) secara otomatis pada model:

```typescript
// app/models/base.model.ts
import { Model, context } from "@utils";

export class BaseModel extends Model {
  beforeSave() {
    const currentUserId = context.get("user_id");
    if (currentUserId) {
      if (!this.id) {
        this.created_by = currentUserId;
      }
      this.updated_by = currentUserId;
    }
  }
}
```
*Catatan untuk Agen: Gunakan `context.get("user_id")` untuk melacak pembuat atau pengubah data di tingkat ORM secara aman.*
