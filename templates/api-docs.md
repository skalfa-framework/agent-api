# Dokumentasi API & Standar Query (Backend)

Berkas ini mendokumentasikan rute API yang tersedia beserta aturan parameter bawaan dan perilaku otomatis dari framework Skalfa API.

## 1. Perilaku Otomatis (Magic Behaviors) Framework

### A. Resolve Query (`Model.query().resolve(c)`)
Metode `.resolve(c)` otomatis mengikat parameter query HTTP dari `ControllerContext` (`c`) ke query builder database. 

Setiap endpoint yang menggunakan `.resolve(c)` otomatis mendukung parameter query berikut dari client:
*   `page`: (number) Halaman aktif untuk paginasi (misal: `?page=1`).
*   `paginate`: (number) Jumlah baris data per halaman (misal: `?paginate=15`).
*   `search`: (string) Kata kunci pencarian global. Kolom yang dicari ditentukan oleh properti `searchable` pada model.
*   `filter`: (object) Filter spesifik kolom (misal: `?filter[status]=pending`).
*   `expand`: (array/string) Memuat data relasi secara eager loading (misal: `?expand[]=role` atau `?expand=role`).
*   `sort`: (string) Kolom pengurutan (misal: `?sort=created_at` or `?sort=-created_at` untuk descending).

### C. Mode Pilihan / Dropdown (Options Mode via `x-option` / `x-options`)
Jika client mengirimkan HTTP header `x-option` (atau `x-options` bernilai `"true"`, `"1"`, atau `"yes"`) atau query parameter `isOption=true`, query builder yang di-resolve menggunakan `.resolve(c)` otomatis berubah menjadi **Options Mode**:
*   **Format Output**: Mengembalikan data dalam bentuk pasangan `value` dan `label` yang siap digunakan untuk komponen select/dropdown:
    ```json
    {
      "data": [
        { "value": 1, "label": "Nama Item A" },
        { "value": 2, "label": "Nama Item B" }
      ],
      "total": 2
    }
    ```
*   **Pemetaan Kolom**: Secara default, `value` dipetakan ke kolom pertama pada properti `selectable` model (atau `primaryKey`), dan `label` dipetakan ke kolom kedua. Kustomisasi kolom dapat dikirim melalui query parameter `option` (contoh: `?option[]=id&option[]=title`).

*Catatan untuk Agen: Jangan mengimplementasikan ulang logika paginasi, pencarian, pengurutan, filter, atau pembuatan opsi dropdown secara manual jika controller sudah menggunakan `.resolve(c)`.*


### B. Pump Data (`Model.pump(payload, options)`)
Metode `.pump()` adalah mesin penyimpanan data transaksional otomatis yang mendukung penyimpanan bersarang (nested relation) dan massal (bulk):
*   **Pemetaan Otomatis**: Memisahkan key pada `payload` menjadi kolom lokal (yang terdaftar sebagai `fillable` pada model) dan data relasi (yang terdaftar sebagai `relations`).
*   **Penyimpanan Relasi Otomatis**:
    *   Jika relasi berupa objek tunggal (`belongsTo`, `hasOne`), `.pump()` otomatis membuat/memperbarui data anak tersebut.
    *   Jika relasi berupa array (`hasMany`, `belongsToMany`), `.pump()` otomatis melakukan sinkronisasi: membuat data baru, memperbarui data yang sudah ada, dan **menghapus** data lama yang tidak disertakan dalam array input (orphan removal).
*   **Bulk Mode**: Jika `payload` berbentuk array, `.pump()` otomatis mengulang proses untuk setiap item dalam satu transaksi database.

---

## 2. Daftar Endpoint API

*(Daftar endpoint API akan ditambahkan dan diperbarui di sini oleh agen saat mengembangkan fitur baru)*
