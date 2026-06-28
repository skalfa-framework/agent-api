# Panduan Utilitas: Aturan Validasi (Validation) (`@utils`)

Dokumen ini menjelaskan penggunaan mesin validasi bawaan Skalfa API untuk menvalidasi request payload di sisi backend.

## 1. Menjalankan Validasi di Controller

Validasi dipicu di dalam method controller menggunakan `await c.validation({ ... })` sebelum memproses data. Jika validasi gagal, framework secara otomatis melempar error `422 Unprocessable Entity` dan mengembalikan daftar error terperinci ke client.

```typescript
import { ControllerContext } from "@utils";

export class AuthController {
  static async register(c: ControllerContext) {
    // Jalankan validasi
    await c.validation({
      name:                  ["required", "string", "max:100"],
      email:                 ["required", "email", "unique:users,email"],
      password:              ["required", "min:8", "confirmed"],
      password_confirmation: ["required"]
    });

    // Data aman untuk diproses...
  }
}
```

---

## 2. Daftar Aturan Validasi (Validation Rules)

Skalfa API mendukung aturan validasi berikut:

### A. Validasi Tipe Data & Keberadaan
*   `required`: Data wajib diisi dan tidak boleh kosong.
*   `string` / `text`: Data harus berupa string.
*   `numeric` / `number`: Data harus berupa angka/numerik.
*   `boolean`: Data harus bernilai boolean (`true`, `false`, `"true"`, `"false"`, `1`, `0`).
*   `array`: Data harus berupa array.
*   `email`: Data harus berupa format email yang valid.
*   `url`: Data harus berupa URL yang valid.
*   `date`: Data harus berupa tanggal yang valid.

### B. Validasi Panjang & Batas (Length)
*   `min:N`: Panjang string minimal `N` karakter (contoh: `min:8`).
*   `max:N`: Panjang string maksimal `N` karakter (contoh: `max:100`).
*   `between:MIN,MAX`: Panjang string harus berada di antara `MIN` dan `MAX` karakter (contoh: `between:3,20`).

### C. Validasi Keanggotaan Set (Set Membership)
*   `in:VAL1,VAL2,...`: Nilai harus salah satu dari daftar yang ditentukan (contoh: `in:active,pending,failed`).
*   `not_in:VAL1,VAL2,...`: Nilai tidak boleh berupa salah satu dari daftar yang ditentukan (contoh: `not_in:admin,superadmin`).

### D. Validasi Hubungan Antar Field (Relational)
*   `confirmed`: Nilai field harus cocok dengan field konfirmasinya, yaitu `${field}_confirmation` (contoh: `password` akan dicocokkan dengan `password_confirmation`).
*   `same:FIELD_NAME`: Nilai field harus sama persis dengan nilai field `FIELD_NAME`.
*   `different:FIELD_NAME`: Nilai field harus berbeda dengan nilai field `FIELD_NAME`.
*   `regex:PATTERN`: Nilai field harus cocok dengan pola ekspresi reguler (regular expression) (contoh: `regex:^[A-Z0-9]+$`).

### E. Validasi Database (Database Validation)
Aturan ini langsung melakukan pengecekan ke database secara otomatis:
*   `unique:TABLE,COLUMN,EXCEPT_ID`: Nilai harus unik (belum digunakan) di tabel `TABLE` kolom `COLUMN`.
    *   *Pengecualian*: Jika memperbarui data lama, kirimkan ID data tersebut di parameter ketiga agar tidak bentrok dengan dirinya sendiri (contoh: `unique:users,email,${userId}`).
    *   *Soft Delete*: Validasi ini secara otomatis mengabaikan baris yang memiliki `deleted_at IS NOT NULL` (menghormati soft delete).
*   `exists:TABLE,COLUMN`: Nilai harus ada (terdaftar) di tabel `TABLE` kolom `COLUMN`.
    *   *Soft Delete*: Secara otomatis hanya mencari baris yang memiliki `deleted_at IS NULL`.
