# Utility Guide: Logging (`logger`)

The `logger` utility provides a structured logging system to track application events, SQL queries, and error stack traces.

---

## 1. Log Levels

Supported log levels:
*   `info`: General operational messages.
*   `warn`: Warnings about potential issues.
*   `error`: Critical errors that require attention.
*   `debug`: Detailed debugging information.

---

## 2. Usage

You can import the `logger` utility to log events from anywhere in the application:

```typescript
import { logger } from '@utils'

// Logging info
logger.info("Server started successfully on port 3000");

// Logging warnings
logger.warn("Redis connection lost. Retrying...", { attempt: 3 });

// Logging errors with context and stack trace
try {
  throw new Error("Database connection timed out");
} catch (err) {
  logger.error("Failed to connect to database", err);
}
```

---

## 3. Log Output

Logs are written to two destinations:
1.  **Console**: Formatted for readability in development.
2.  **Log File**: Saved under `storage/logs/` as JSON lines (e.g. `storage/logs/error.log` and `storage/logs/combined.log`) in production.
