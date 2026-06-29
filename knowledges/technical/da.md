# Technical Guide: Data Analytics & OLAP (`da`)

For analytical queries, reporting, and big data processing, Skalfa API integrates with ClickHouse (an OLAP database) via the `da` (Data Analytics) utility.

---

## 1. Why Separate OLTP and OLAP?

*   **OLTP (PostgreSQL/MySQL)**: Optimized for fast transaction handling, row-level updates, and strict consistency. Not suitable for scanning millions of rows for reports.
*   **OLAP (ClickHouse)**: Optimized for column-based storage, high compression, and extremely fast aggregation queries across billions of rows. Writes should be done in batches.

---

## 2. Writing Analytics Data

Writing to ClickHouse should be done using batch inserts. Avoid inserting row-by-row as it creates too many parts on the ClickHouse storage engine.

```typescript
// app/controllers/analytics/event.controller.ts
import { ControllerContext } from 'elysia'
import { da } from '@utils'

export class EventController {
  static async track(c: ControllerContext) {
    await c.validation({
      event_name: ["required"],
      metadata:   ["required", "json"]
    })

    // Batch insert event
    await da.insert("user_events", [
      {
        event_id:   Date.now().toString(36),
        event_name: c.payload.event_name,
        user_id:    c.user?.id || 0,
        metadata:   JSON.stringify(c.payload.metadata),
        created_at: new Date()
      }
    ])

    c.responseSuccess(null, "Event tracked successfully")
  }
}
```

---

## 3. Querying Analytics Data

Querying ClickHouse returns raw data rows. Use standard SQL aggregations.

```typescript
// app/controllers/analytics/report.controller.ts
import { ControllerContext } from 'elysia'
import { da } from '@utils'

export class ReportController {
  static async getSummary(c: ControllerContext) {
    const rows = await da.query(`
      SELECT 
        event_name, 
        count() as total_count 
      FROM user_events 
      GROUP BY event_name 
      ORDER BY total_count DESC
    `)

    c.responseData(rows)
  }
}
```
