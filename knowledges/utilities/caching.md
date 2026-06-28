# Panduan Utilitas: Caching (Redis) (`@utils`)

Skalfa API menyediakan utilitas caching berbasis Redis (`ioredis`) untuk menyimpan data sementara, mengurangi beban query database berulang, dan mempercepat respons API.

---

## 1. Koneksi Redis

Koneksi Redis dikonfigurasi melalui berkas `.env` dan diinisialisasi secara global sekali untuk seluruh aplikasi:
```text
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0
```

---

## 2. Membuat Kunci Cache (`cache.makeKey`)

Untuk memastikan format nama kunci cache (*cache keys*) konsisten dan terhindar dari bentrok, gunakan `cache.makeKey`.

```typescript
import { cache } from "@utils";

// Format: cache.makeKey(type, prefix, paramsOrQuery?)
const key = cache.makeKey('db', 'users', {
  page:   1,
  search: 'john'
});

// Hasil: "db:users:page=1&search=john" (atau terenkripsi/hash otomatis di bawah kap)
```

---

## 3. Menyimpan & Mengambil Data Cache

### A. Menyimpan Data (`cache.set`)
Menyimpan data ke Redis dengan waktu kedaluwarsa (TTL) dalam satuan **detik**.

```typescript
import { cache } from "@utils";

const users = [{ id: 1, name: "John Doe" }];

// Simpan ke cache selama 60 detik
await cache.set(key, users, 60);
```

### B. Mengambil Data (`cache.get`)
Membaca data dari cache. Mengembalikan data ter-parsing jika valid, atau `null` jika cache tidak ditemukan atau telah kadaluwarsa.

```typescript
import { cache } from "@utils";

const cachedData = await cache.get<User[]>(key);

if (cachedData) {
  return cachedData; // Cache hit
}
```

---

## 4. Pola Kerja Cache-Aside (Praktek Terbaik)

Pola paling umum untuk meningkatkan performa pembacaan data:
1.  Periksa apakah data ada di cache.
2.  Jika ada (*cache hit*), langsung kembalikan data tersebut.
3.  Jika kosong (*cache miss*), ambil data terbaru dari database.
4.  Simpan data tersebut ke cache dengan TTL yang sesuai untuk request berikutnya.

```typescript
import { cache, ControllerContext } from "@utils";
import { User } from "@models";

static async index(c: ControllerContext) {
  const key = cache.makeKey('db', 'users', c.getQuery);

  let users = await cache.get(key);

  if (!users) {
    // Cache miss - Ambil dari DB
    users = await User.query().resolve(c);
    
    // Simpan ke cache selama 120 detik
    await cache.set(key, users, 120);
  }

  c.responseData(users.data, users.total);
}
```

---

## 5. Membersihkan Cache (`cache.clear`)

Wajib menghapus cache terkait setelah terjadi modifikasi data (seperti menambah, mengubah, atau menghapus data) agar pengguna selalu melihat data terbaru.

```typescript
import { cache } from "@utils";

// Menghapus seluruh cache dengan tipe 'db' dan prefix 'users'
await cache.clear('db', 'users');
```
