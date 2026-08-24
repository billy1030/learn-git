# Zeabur PostgreSQL Troubleshooting & Investigation Guide

This guide documents the root causes, diagnostic steps, and solutions for PostgreSQL-related issues encountered when running the **Material Data System (MDS)** on the **Zeabur Cloud Container Platform**.

---

## 1. Issue: `Internal Server Error (500)` During Material Creation

### 1.1. Symptom
* When submitting the **Create New Material** form on Zeabur (`https://mds.zeabur.app`), the UI displayed a red alert: `Internal server error`.
* The server responded with HTTP status `500 Internal Server Error`.

### 1.2. Root Cause Analysis
1. **Invalid Date Format in SQL Query (`production_month` concatenation bug)**:
   - In HTML5, `<input type="date">` submits ISO format `YYYY-MM-DD` (e.g. `2026-08-19`).
   - The backend service code previously contained:
     ```typescript
     // BUG: Appended "-01" unconditionally
     production_month: input.production_month ? `${input.production_month}-01` : null
     ```
   - When a full date was passed, the resulting string became `2026-08-19-01`.
   - PostgreSQL threw a `22007: invalid input syntax for type date: "2026-08-19-01"` runtime exception inside the transactional query, causing Fastify to return `500`.

2. **Missing Schema Fields in Insert Statement**:
   - `frozen` and `freeze_reason` fields were absent in the insert payload for `createMaterial()`.

### 1.3. Solution & Code Fix
In [`backend/src/services/materials.service.ts`](file:///c:/ai/mds/backend/src/services/materials.service.ts):
```typescript
let formattedProductionMonth: string | null = null;
if (input.production_month) {
  if (input.production_month.length === 7) {
    // Input is 'YYYY-MM' -> normalize to first day of month
    formattedProductionMonth = `${input.production_month}-01`;
  } else {
    // Input is already 'YYYY-MM-DD'
    formattedProductionMonth = input.production_month;
  }
}

const inserted = await tx
  .insert(materials)
  .values({
    serial: input.serial,
    production_batch: input.production_batch,
    batch_quantity: input.batch_quantity ?? null,
    name: input.name,
    manufacturer: input.manufacturer,
    production_month: formattedProductionMonth,
    delivery_date: input.delivery_date || null,
    sample_tested_flag: input.sample_tested_flag ?? false,
    frozen: input.frozen ?? false,
    freeze_reason: input.freeze_reason || null,
    version: 1,
  })
  .returning();
```

---

## 2. Issue: `Invalid request data` Validation Rejections

### 2.1. Symptom
* Submitting test materials with shorter serial numbers (e.g. `test` or `B001`) was rejected before reaching PostgreSQL with the error `Invalid request data`.

### 2.2. Root Cause Analysis
* The backend Zod validation schema had a strict constraint:
  ```typescript
  serial: z.string().min(8, "serial must be at least 8 characters")
  ```
  While the frontend form only enforced `.min(1)`, leading to mismatched validation expectations between client and server.

### 2.3. Solution & Code Fix
In [`backend/src/schemas/material.schema.ts`](file:///c:/ai/mds/backend/src/schemas/material.schema.ts):
```typescript
export const CreateMaterialBodySchema = z.object({
  serial: z.string().min(1, "serial is required").max(64),
  production_batch: z.string().min(1, "production_batch is required").max(64),
  // ...
});
```
In [`frontend/src/components/admin/MaterialCreateForm.tsx`](file:///c:/ai/mds/frontend/src/components/admin/MaterialCreateForm.tsx):
Improved the error banner to extract and display nested Zod error details (`details[0].message`) directly so operators understand the exact field that failed validation.

---

## 3. Issue: Database CHECK Constraint Violations on Tracking Events

### 3.1. Symptom
* PostgreSQL error code `23514`:
  ```text
  new row for relation "tracking_events" violates check constraint "tracking_events_event_type_check"
  ```

### 3.2. Root Cause Analysis
* In [`0003_phase2_inspections_events.sql`](file:///c:/ai/mds/backend/src/db/migrations/0003_phase2_inspections_events.sql), the `CHECK` constraint originally only permitted:
  `CHECK ("event_type" IN ('出廠', '運送', '抽樣', '檢測', '配送'))`
* New business lifecycle events like `進場` (Site Ingress) and `安裝` (Installation) were rejected by PostgreSQL.

### 3.3. Solution & Code Fix
Updated the database schema definition and migration constraint:
```sql
CONSTRAINT "tracking_events_event_type_check" 
  CHECK ("event_type" IN ('出廠', '運送', '抽樣', '檢測', '配送', '進場', '安裝'))
```

---

## 4. Zeabur Cloud PostgreSQL Connection Best Practices

### 4.1. Internal vs. External Connection String
Always use the **Internal Connection String** inside the Zeabur network:
* **Internal**: `postgresql://postgres:<PASSWORD>@postgresql.zeabur.internal:5432/<DB_NAME>` (Zero network latency, encrypted internal bridge, no bandwidth charges)
* **Public/External**: Used only for local tools like DBeaver / pgAdmin / DataGrip.

### 4.2. Connection Pool Tuning for Container Idle / Sleep
When containers become idle on cloud platforms, TCP sockets may silently drop:
```typescript
// backend/src/db/client.ts
pool = new Pool({
  connectionString: url,
  max: 10,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 5000,
  keepAlive: true,
});
```

---

## 5. Troubleshooting Checklist & Quick Reference

| Issue | Diagnostic Command / Step | Resolution |
|---|---|---|
| **500 Internal Server Error** | Check Zeabur **Runtime Logs** for SQL syntax / constraint code. | Fix schema mismatch or date parsing in backend service. |
| **400 Invalid Request Data** | Inspect HTTP payload in browser Network tab against `material.schema.ts`. | Check Zod constraints (min/max length, regex formats). |
| **23505 Duplicate Serial** | Run `SELECT serial FROM materials WHERE serial = '...';` | Serial numbers must be unique across all active and frozen items. |
| **Zeabur Serving Stale Code** | Check latest commit hash in Zeabur **Deployments** tab. | Trigger **Redeploy with Clean Cache** and hard refresh browser (`Ctrl + F5`). |

---

## 6. Comprehensive Codebase Audit & Pattern Verification

A complete scan of all services, schemas, and migrations was performed to verify if any other modules had similar date parsing, schema mismatches, or constraint issues:

| Module / File | Verified Area | Status | Audit Findings |
|---|---|---|---|
| [`materials.service.ts`](file:///c:/ai/mds/backend/src/services/materials.service.ts) | `updateMaterial()` | ✅ Verified & Hardened | Already checks `if (patch.production_month.length === 7)` before appending `-01`. Correctly handles both `YYYY-MM` and `YYYY-MM-DD`. |
| [`inspections.service.ts`](file:///c:/ai/mds/backend/src/services/inspections.service.ts) | `upsertInspection()` | ✅ Clean | Directly accepts ISO strings or `YYYY-MM-DD` and stores them via Drizzle ORM without manual string concatenation. |
| [`tracking-events.service.ts`](file:///c:/ai/mds/backend/src/services/tracking-events.service.ts) | `createEvent()` / `updateEvent()` | ✅ Clean | Directly parses timestamptz dates via `new Date(body.event_date)`. |
| [`materials.service.ts`](file:///c:/ai/mds/backend/src/services/materials.service.ts) | `exportMaterialsForExcel()` | ✅ Clean | Uses explicit SQL type casting (`${filters.startDate}::timestamp` and `(${filters.endDate}::date + interval '1 day')`) preventing database syntax errors. |
| [`backup.service.ts`](file:///c:/ai/mds/backend/src/services/backup.service.ts) | Backup Snapshot & Restore Engine | ✅ Clean | Restores entity records and parses dates using `row.created_at ? new Date(row.created_at) : new Date()`. |
| [`material.schema.ts`](file:///c:/ai/mds/backend/src/schemas/material.schema.ts) | Zod `CreateMaterialBodySchema` | ✅ Aligned | `serial` minimum length updated to `.min(1)` so test and production serial numbers validate consistently without `Invalid request data` errors. |
| [`0003_phase2_inspections_events.sql`](file:///c:/ai/mds/backend/src/db/migrations/0003_phase2_inspections_events.sql) | SQL `CHECK` Constraints | ✅ In-Sync | Enums for `tracking_events`, `inspections`, and `users` are 100% aligned with application Zod schemas. |

