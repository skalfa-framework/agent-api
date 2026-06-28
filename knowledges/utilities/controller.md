# Panduan Utilitas: Pengendali (Controller Utility) (`@utils`)

Controller Utility di Skalfa API menyediakan kumpulan helper yang disuntikkan langsung ke dalam objek `ControllerContext` (`c`). Tujuannya adalah menyeragamkan cara membaca request, melakukan validasi, serta mengembalikan response tanpa menulis ulang boilerplate di setiap controller.

---

## 1. Pengambilan Parameter Query (`c.getQuery`)

Membaca, menormalkan, dan mengonversi tipe data query string dari URL secara otomatis.

```typescript
// URL: /api/users?page=2&paginate=15&search=budi&sort=["name desc"]
static async index(c: ControllerContext) {
  const { page, paginate, search, sort, filter, expand } = c.getQuery;
  
  // page = 2 (number)
  // paginate = 15 (number)
  // search = "budi" (string)
  // sort = ["name desc"] (array string)
}
```

---

## 2. Validasi Payload Request (`c.validation`)

Memvalidasi data `c.body` berdasarkan aturan (*rules*). Jika validasi gagal, eksekusi dihentikan seketika dan server otomatis mengembalikan status `422 Unprocessable Entity` beserta detail error.

```typescript
static async store(c: ControllerContext) {
  await c.validation({
    name:  'required|string|max:100',
    email: 'required|email|unique:users,email'
  });

  // Data aman untuk diproses...
}
```

---

## 3. Helper Pengiriman Respons

Semua helper respons di bawah ini langsung menghentikan proses eksekusi (*early return/throw*) dengan status HTTP yang sesuai untuk menjaga konsistensi format JSON API Skalfa.

### A. Respons Daftar Data (`c.responseData`)
Mengembalikan status `200 OK` dengan format data list dan total baris untuk paginasi.
```typescript
c.responseData(users, total);
// JSON: { "message": "Success", "data": [...], "total_row": 50 }
```

### B. Respons Sukses Umum (`c.responseSuccess`)
Mengembalikan status `200 OK` (atau `201` jika diatur) untuk aksi non-create (seperti update, delete, atau aksi proses).
```typescript
c.responseSuccess(record, 'Data berhasil diperbarui');
```

### C. Respons Simpan Data (`c.responseSaved`)
Mengembalikan status `201 Created` setelah sukses menyimpan data baru.
```typescript
c.responseSaved(record);
```

### D. Respons Forbidden (`c.responseForbidden`)
Mengembalikan status `403 Forbidden` saat otorisasi atau hak akses ditolak.
```typescript
c.responseForbidden("Anda tidak memiliki akses ke fitur ini");
```

### E. Respons Gagal Server-side (`c.responseError`)
Mencatat detail error ke Winston logger dan melempar status `500 Internal Server Error`.
```typescript
try {
  // logic...
} catch (err) {
  c.responseError(err as Error, 'User.store', 'Gagal menyimpan data user');
}
```

---

## 4. Helper Manajemen Berkas

*   **`c.uploadFile(file, folder?)`**: Mengunggah berkas ke disk storage dan mengembalikan path relatifnya.
    ```typescript
    const path = await c.uploadFile(c.body.file, 'avatars');
    ```
*   **`c.deleteFile(filePath)`**: Menghapus berkas fisik dari storage.
    ```typescript
    c.deleteFile('storage/public/avatars/example.png');
    ```
