# Panduan Utilitas: Enkripsi Data (Encrypting) (`@utils`)

Utilitas `encryption` menyediakan metode untuk melakukan enkripsi dan dekripsi data menggunakan berbagai algoritma keamanan. Berguna untuk mengamankan data sensitif sebelum disimpan ke database/cookie.

---

## 1. Algoritma Dua Arah (AES & TripleDES)

Algoritma dua arah digunakan untuk data yang perlu diamankan tetapi harus bisa didekripsi kembali ke bentuk aslinya (seperti access token, data pribadi).

### A. Enkripsi AES (`encryption.set`)
Mengenkripsi string atau objek menggunakan AES (default). Jika input berupa objek, otomatis diubah menjadi format JSON sebelum dienkripsi.

```typescript
import { encryption } from "@utils";

// 1. Enkripsi String
const encryptedStr = encryption.set("Hello World"); 
// Hasil: Base64 string terenkripsi

// 2. Enkripsi Objek
const payload = { id: 1, name: "Ahmad" };
const encryptedObj = encryption.set(payload);
```

### B. Dekripsi AES (`encryption.get`)
Mengembalikan data terenkripsi ke bentuk aslinya. Jika data aslinya berupa JSON objek, otomatis dikembalikan menjadi objek kembali.

```typescript
import { encryption } from "@utils";

// 1. Dekripsi String
const decryptedStr = encryption.get(encryptedStr); // Hasil: "Hello World"

// 2. Dekripsi Objek
const data = encryption.get(encryptedObj); // Hasil: { id: 1, name: "Ahmad" }
```

---

## 2. Algoritma Satu Arah / Hash (SHA256, SHA512, MD5)

Algoritma satu arah digunakan untuk validasi data atau pembuatan sidik jari berkas. Data hasil enkripsi satu arah **tidak bisa didekripsi kembali**.

```typescript
import { encryption } from "@utils";

// Pembuatan Hash Satu Arah
const shaHash = encryption.set("Hello", "", "SHA256");
const md5Hash = encryption.set("Hello", "", "MD5");

// Catatan: Mencoba melakukan dekripsi pada hash akan melempar error
// encryption.get(shaHash, "", "SHA256"); // ❌ ERROR!
```

---

## 3. Kunci Rahasia Kustom (Custom Secret Key)
Secara default, utilitas menggunakan kunci rahasia aplikasi. Anda dapat mengirimkan kunci rahasia kustom pada parameter kedua:

```typescript
const encrypted = encryption.set("Hello", "kunci-rahasia-kustom", "AES");
const decrypted = encryption.get(encrypted, "kunci-rahasia-kustom", "AES");
```
*Catatan untuk Agen: Selalu gunakan `encryption.set` untuk mengenkripsi token atau payload sensitif sebelum disimpan ke dalam cookie.*
