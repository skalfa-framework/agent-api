# Utility Guide: Data Formatting & Converting (`converting`)

The `converting` utility provides formatters for currency, dates, and localized values.

---

## 1. Currency Formatter (`currency`)

Formats numbers into localized currency strings. By default, it formats to Indonesian Rupiah (`Rp`).

```typescript
import { conversion } from '@utils'

// Default: IDR (Rupiah)
conversion.currency(15000);
// => "Rp15.000"

// Custom locale and currency code
conversion.currency(100, "en-US", "USD");
// => "$100.00"
```

---

## 2. Date Formatter (`date`)

Formats dates using standard tokens. It uses native `Intl.DateTimeFormat` with Indonesian locale (`id-ID`) under the hood.

```typescript
import { conversion } from '@utils'

// Default format: "DD MMM YYYY"
conversion.date("2026-06-29");
// => "29 Jun 2026"

// Custom format
conversion.date("2026-06-29", "YYYY/MM/DD");
// => "2026/06/29"

// Format with day name
conversion.date("2026-06-29", "dddd, DD MMMM YYYY");
// => "Senin, 29 Juni 2026"
```

### Supported Tokens:
*   `YYYY` / `YY`: 4 or 2 digit year.
*   `MMMM` / `MMM` / `MM` / `M`: Month name (full, short), 2 digit, or 1 digit month.
*   `DD` / `D`: 2 or 1 digit day.
*   `dddd` / `ddd`: Day name (full, short).
*   `HH` / `H` / `hh` / `h`: Hours (24h or 12h formats).
*   `mm` / `m`: Minutes.
*   `ss` / `s`: Seconds.
