# Panduan Utilitas: Hak Akses & Pengaman (Permission) (`@utils`)

Utilitas `permission` digunakan untuk mendaftarkan modul fitur beserta hak aksesnya (permissions) dan mengamankan metode di controller menggunakan guard.

---

## 1. Pendaftaran Izin (`permission.register`)

Pendaftaran izin dilakukan di tingkat paling atas berkas controller. Anda mendefinisikan kode fitur 3 digit (`KeyFeature`, misal: `"100"`) dan daftar aksi akses 2 digit (`KeyAccess`, misal: `"01"`, `"02"`).

```typescript
import { permission } from "@utils";

export const UserPermission = permission.register({
  '100': {
    name: 'User Management',
    accesses: {
      '01': 'View',
      '02': 'Create',
      '03': 'Update',
      '04': 'Delete',
    }
  },
});
```

---

## 2. Pengamanan Metode Controller (`guard`)

Gunakan objek permission yang terdaftar untuk mengamankan metode controller. Jika user yang sedang login tidak memiliki hak akses tersebut, guard otomatis memanggil `c.responseForbidden()` dan menghentikan request dengan status `403`.

### A. Pengaman Tunggal (Single Permission)
```typescript
static async index(c: ControllerContext) {
  // Memeriksa izin "100.01" (View)
  UserPermission.have('01').guard(c);

  const users = await User.query().resolve(c);
  c.responseData(users.data, users.total);
}
```

### B. Pengaman Kombinasi (OR Logic via `orHave`)
Jika metode dapat diakses apabila pengguna memiliki salah satu dari beberapa izin.

```typescript
static async store(c: ControllerContext) {
  // Memeriksa apakah user memiliki izin "100.02" (Create) ATAU "100.03" (Update)
  UserPermission.have('02').orHave('03').guard(c);

  // Proses simpan...
}
```

### C. Pengaman Lintas Fitur (Cross-Feature)
Jika Anda perlu memeriksa izin fitur lain yang tidak terikat pada modul controller saat ini, tuliskan kode izin lengkap dengan pemisah titik (`.`).

```typescript
// Memeriksa izin modul lain (misal modul "200" akses "01")
UserPermission.have('200.01').guard(c);
```
*Catatan untuk Agen: Semua pengamanan wajib ditulis di baris pertama di setiap method controller sebelum logika bisnis apa pun dijalankan.*
