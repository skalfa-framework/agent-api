# Panduan Utilitas: Rute CRUD Otomatis (Route) (`@utils`)

Utilitas `route` digunakan untuk menyederhanakan pendaftaran endpoint API di Skalfa API. Jika controller Anda mengikuti pola RESTful standar (memiliki metode `index`, `store`, `show`, `update`, dan `destroy`), Anda dapat mendaftarkan kelima rute tersebut sekaligus dalam satu baris menggunakan `route.api`.

## 1. Pemetaan Rute Otomatis (`route.api`)

Fungsi `route.api(app, basePath, controller)` secara otomatis mendaftarkan rute grup berikut di Elysia:

| Method HTTP | Path Rute | Method Controller | Aksi / Tujuan |
|-------------|-----------|-------------------|---------------|
| `GET`       | `/`       | `controller.index` | Mengambil daftar data (list) |
| `POST`      | `/`       | `controller.store` | Menyimpan data baru (create) |
| `GET`       | `/:id`    | `controller.show`  | Mengambil satu detail data |
| `PUT`       | `/:id`    | `controller.update`| Memperbarui data lama |
| `DELETE`    | `/:id`    | `controller.destroy`| Menghapus data (soft delete) |

---

## 2. Contoh Penggunaan di Rute Utama (`app/routes.ts`)

```typescript
import { Elysia } from "elysia";
import { route } from "@utils";
import { UserController } from "@controllers";

export const apiRoutes = (app: Elysia) => {
  // Otomatis mendaftarkan 5 endpoint RESTful di bawah path "/users"
  return route.api(app, "/users", UserController);
};
```

---

## 3. Menambahkan Rute Kustom Tambahan
Jika sebuah modul memerlukan endpoint khusus di luar 5 aksi CRUD standar, Anda dapat mendaftarkannya secara manual menggunakan grup Elysia biasa:

```typescript
export const bookingRoutes = (app: Elysia) => {
  return app.group("/bookings", (group) => group
    // 1. Daftarkan rute khusus terlebih dahulu
    .put("/:id/approve", BookingController.approve)
    
    // 2. Daftarkan rute CRUD otomatis setelahnya
    .use((g) => route.api(g, "/", BookingController))
  );
};
```
*Catatan untuk Agen: Selalu gunakan `route.api` untuk pendaftaran rute CRUD standar guna mengurangi baris kode boilerplate rute di Elysia.*
