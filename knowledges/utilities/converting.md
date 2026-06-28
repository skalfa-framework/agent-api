# Panduan Utilitas: Konversi Format (Converting) (`@utils`)

Utilitas `conversion` di backend menyediakan metode pemformatan yang identik dengan frontend untuk menjaga konsistensi manipulasi string, nominal mata uang, dan tanggal di kedua sisi aplikasi Skalfa.

---

## 1. Manipulasi Huruf String

*   **Snake Case (`strSnake`)**:
    Mengubah string menjadi format snake_case.
    ```typescript
    conversion.strSnake("HelloWorld"); // Hasil: "hello_world"
    ```
*   **Slug Case (`strSlug`)**:
    Mengubah string menjadi format kebab-case/url-slug.
    ```typescript
    conversion.strSlug("Hello World"); // Hasil: "hello-world"
    ```
*   **Camel Case (`strCamel`)**:
    Mengubah string menjadi format camelCase.
    ```typescript
    conversion.strCamel("hello_world"); // Hasil: "helloWorld"
    ```
*   **Pascal Case (`strPascal`)**:
    Mengubah string menjadi format PascalCase.
    ```typescript
    conversion.strPascal("hello_world"); // Hasil: "HelloWorld"
    ```
*   **Bentuk Jamak (`strPlural`)**:
    Mengubah kata benda bahasa Inggris menjadi bentuk jamak.
    ```typescript
    conversion.strPlural("category"); // Hasil: "categories"
    ```
*   **Bentuk Tunggal (`strSingular`)**:
    Mengubah kata benda menjadi bentuk tunggal (kapitalisasi Pascal).
    ```typescript
    conversion.strSingular("booking_payments"); // Hasil: "BookingPayments"
    ```

---

## 2. Pemformatan Mata Uang (`currency`)

Memformat angka nominal menjadi mata uang Rupiah secara standar.

```typescript
import { conversion } from "@utils";

conversion.currency(150000); // Hasil: "Rp 150.000"
```

---

## 3. Pemformatan Tanggal Lokal (`date`)

Memformat string tanggal menggunakan standardisasi `Intl.DateTimeFormat` lokalisasi Indonesia (`id-ID`).

```typescript
import { conversion } from "@utils";

const isoDate = "2026-06-28T12:00:00.000Z";

// Format default: "DD MMM YYYY"
conversion.date(isoDate); // Hasil: "28 Jun 2026"

// Format kustom jam dan menit
conversion.date(isoDate, "YYYY-MM-DD HH:mm"); // Hasil: "2026-06-28 12:00"
```
