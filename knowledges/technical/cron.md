# Technical Guide: Scheduled Tasks (`cron`)

Cron jobs in Skalfa API are used to run background tasks periodically. They are managed via the `cron` utility.

---

## 1. Defining Cron Jobs

Cron jobs are registered in `app/jobs/crons/` and must be imported in the main entry point to start.

```typescript
// app/jobs/crons/cleanup.cron.ts
import { cron, db } from '@utils'

// Runs every day at midnight
cron.add("cleanup-expired-tokens", "0 0 * * *", async () => {
  console.log("🧹 Running token cleanup...");
  
  const threshold = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000); // 30 days ago
  await db("personal_access_tokens")
    .where("last_used_at", "<", threshold)
    .delete();
    
  console.log("✅ Token cleanup completed.");
})
```

---

## 2. Cron Expression Format

Standard 5-field cron syntax is supported:
```text
* * * * *
│ │ │ │ │
│ │ │ │ └─ Day of the week (0 - 6) (Sunday to Saturday)
│ │ │ └─ Month (1 - 12)
│ │ └─ Day of the month (1 - 31)
│ └─ Hour (0 - 23)
└─ Minute (0 - 59)
```

Common intervals:
*   `*/5 * * * *`: Every 5 minutes.
*   `0 * * * *`: Every hour.
*   `0 0 * * *`: Every day at midnight.
*   `0 0 * * 0`: Every Sunday at midnight.

---

## 3. Execution Environment

*   **Development**: Cron jobs run in the same process as the main server when starting with `npm run dev`.
*   **Production**: In production, it is recommended to run cron jobs in a separate worker process using a dedicated command to avoid blocking the main HTTP thread:
    ```bash
    bun run app/jobs/crons/workers.ts
    ```
