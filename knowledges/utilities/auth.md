# Panduan Utilitas: Autentikasi Sesi (Auth) (`@utils`)

Utilitas `auth` digunakan untuk mengelola sesi pengguna, pembuatan JWT token, verifikasi token email, dan revalidasi cache izin di backend Skalfa API.

## 1. Pembuatan & Verifikasi Token Sesi

*   **`auth.createAccessToken(userId, request)`**:
    *   Membuat token sesi baru acak sepanjang 20 karakter.
    *   Mencatat sesi aktif ke dalam tabel `user_sessions` (menyimpan data browser, IP, dan waktu kadaluwarsa).
    *   Mengembalikan objek token terenkripsi.
*   **`auth.verifyAccessToken(token, request)`**:
    *   Memverifikasi validitas token sesi (mencocokkan ke database atau cache sesi aktif).
    *   Mendeteksi jika token sudah kadaluwarsa atau telah dicabut.
*   **`auth.revokeAccessToken(token)`**:
    *   Menghapus sesi token aktif dari tabel `user_sessions` (proses logout).

---

## 2. Token Verifikasi Email & Reset Sandi

*   **`auth.createUserMailToken(userId, email, type)`**:
    *   Membuat token unik sekali pakai untuk keperluan verifikasi email atau reset kata sandi.
    *   Tipe yang didukung: `"verify_email"` atau `"reset_password"`.
*   **`auth.verifyUserMailToken(token, type)`**:
    *   Memverifikasi token email yang dikirimkan oleh pengguna.

---

## 3. Revalidasi Izin Pengguna (Cache Permission)

*   **`auth.revalidateUserPermissions(userId)`**:
    *   Memperbarui cache daftar hak akses (permissions) untuk user tertentu agar langsung sinkron dengan database jika ada perubahan role/hak akses.
*   **`auth.revalidateUserPermissionsByRole(roleId)`**:
    *   Memperbarui cache daftar hak akses untuk seluruh user yang memiliki role tersebut secara massal.

---

## 4. Contoh Penggunaan di Controller

```typescript
import { auth, ControllerContext } from "@utils";

export class AuthController {
  static async login(c: ControllerContext) {
    // ... validasi & pencocokan password ...
    
    // Buat sesi baru
    const { token } = await auth.createAccessToken(user.id, c.request);
    
    c.responseSuccess({ token }, "Login sukses");
  }

  static async logout(c: ControllerContext) {
    const token = c.request.headers.get("Authorization")?.replace("Bearer ", "");
    if (token) {
      await auth.revokeAccessToken(token);
    }
    c.responseSuccess([], "Logout sukses");
  }
}
```
