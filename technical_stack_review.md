# Technical Stack & Reusable Components Analysis (MDS)

This document provides an in-depth architectural and technical review of reusable modules, micro-utilities, security designs, and granular frontend widgets across the **MDS (Material Data System)** codebase. These components are categorized by granularity—ranging from Enterprise/Major Architectural Subsystems down to **Tiny Widgets, Scan UI Primitives, Micro-Helpers, and Pure Utilities**—and evaluated for future project extraction, modularization, or library publishing.

---

## 1. Executive Summary & Component Matrix

| Component / Utility | Scope / Layer | Reusability Score | Key Tech & Standards | Core Strengths |
| :--- | :--- | :---: | :--- | :--- |
| **Two-Factor Auth (TOTP + Recovery)** | Backend Service / Security | ⭐️⭐️⭐️⭐️⭐️ (High) | RFC 6238 TOTP, HMAC Pre-Auth, Argon2id | Replay attack window tracking, brute-force lockout, single-use recovery hashing |
| **Audit Logging & Object Diff Engine** | Backend Lib + DB | ⭐️⭐️⭐️⭐️⭐️ (High) | Drizzle ORM, JSONB / Deep Diff, Postgres | Transactional integrity, automatic key diffing, IP/UserAgent tracking |
| **Optimistic Concurrency & ETag Engine** | Backend & API | ⭐️⭐️⭐️⭐️⭐️ (High) | Version tracking, `SELECT FOR UPDATE`, Weak ETag | Zero lost updates, standard HTTP 409 & 412 handling |
| **Multi-layer Rate Limiting & Lockout** | Backend Middleware / Security | ⭐️⭐️⭐️⭐️ (High) | `@fastify/rate-limit`, sliding window in DB | IP rate-limiting + username-targeted brute force prevention |
| **Safe Chunked File Upload & Storage** | Backend Service | ⭐️⭐️⭐️⭐️ (High) | Node Streams, UUID sharding `{yyyy}/{mm}`, SHA-256 | Stream-based hash calculation, auto directory partitioning, RFC 5987 headers |
| **Server-Side Pagination, Filter & Search** | Backend & DB Query | ⭐️⭐️⭐️⭐️ (High) | Drizzle ORM, ILIKE Escaping, Parallel Count | Sanitized pattern matching (`%_\`), transactional counting, composable filters |
| **Public vs Admin Serializers (RBAC View)** | Backend Serialization | ⭐️⭐️⭐️⭐️ (High) | Pure Functions / TS Mappings | Zero field leaks to public scan, type-safe stripping |
| **Standardized Error Handling Architecture** | Backend Core | ⭐️⭐️⭐️⭐️⭐️ (High) | Fastify Error Hook, Custom `AppError`, Zod | Uniform JSON error responses, production stack masking |
| **Vertical Audit Timeline (`TrackingTimeline`)** | Scan / UI Widget | ⭐️⭐️⭐️⭐️⭐️ (High) | CSS Pseudo-elements, WAI-ARIA | Tier-aware field masking, stage-for-deletion badge & line-through |
| **Tri-State Result Badge (`ResultBadge`)** | Scan / UI Widget | ⭐️⭐️⭐️⭐️ (High) | React + Lucide Icons | Pill design, accessible status for Pass / Fail / Pending |
| **Frozen Security Lock Banner (`FrozenNotice`)** | Scan / UI Widget | ⭐️⭐️⭐️⭐️ (High) | React Router + Lucide | Lock icon badge, status alert container, accessible backlink |
| **Monospace Serial Cell (`SerialRow`)** | Scan / UI Widget | ⭐️⭐️⭐️⭐️ (High) | Font Mono + CSS `select-all` | Break-all wrap, 1-click select for barcode scanner input verification |
| **SVG-to-PNG Canvas Exporter (`QrPanel`)** | Admin UI Widget | ⭐️⭐️⭐️⭐️⭐️ (High) | HTML5 Canvas + XMLSerializer | Client-side 512x512 SVG rasterization and PNG auto-download |
| **Entity Attachment Manager (`FileAttachmentPanel`)** | Admin UI Widget | ⭐️⭐️⭐️⭐️⭐️ (High) | React Query + File Preview Modal | Client-side validation (10MB limit), image/PDF in-browser preview |
| **Inline Table Editor (`UserDisplayNameCell`)** | Admin UI Widget | ⭐️⭐️⭐️⭐️⭐️ (High) | React + React Query Mutation | Hover reveal, Escape/Enter keys, auto-focus, optimistic refresh |
| **Inline Role Dropdown (`UserRoleSelect`)** | Admin UI Widget | ⭐️⭐️⭐️⭐️ (High) | React + React Query Mutation | Direct mutation on change, disabled during flight |
| **Physical Print Label Card (`PrintLabelCard`)** | Print / UI Widget | ⭐️⭐️⭐️⭐️ (High) | CSS Print Media, SVG QR, mm units | Exact 70x35mm physical dimensions, error fallback, badge layouts |
| **Print Preview Modal (`PrintPreviewModal`)** | Print / UI Widget | ⭐️⭐️⭐️⭐️ (High) | WAI-ARIA Dialog, `window.print()` | A4 2x4 grid layout preview, partial error tally banner |
| **Language Switcher Widget (`LanguageSwitcher`)** | Core UI Widget | ⭐️⭐️⭐️⭐️ (High) | `react-i18next`, Lucide Icons | Accessible `sr-only` label, custom styled select, instant switch |
| **Accessible Dialog Primitive (`ConfirmDialog`)** | Core UI Widget | ⭐️⭐️⭐️⭐️ (High) | React, TailwindCSS, Lucide, WAI-ARIA | Auto-focus, Escape key capture, backdrop blur |
| **Toast Notification Primitive (`Toast`)** | Core UI Widget | ⭐️⭐️⭐️⭐️ (High) | React, CSS Animations, WAI-ARIA | Self-dismissing timer, `aria-live="polite"`, status themes |
| **Axios Auth Interceptor & Router Shield** | Frontend API Client | ⭐️⭐️⭐️⭐️ (High) | Axios Interceptors | Route-aware 401 redirection (bypasses public scan routes `/m/*`) |
| **Pattern Escaping Utility (`escapeLikePattern`)** | Backend Micro-Helper | ⭐️⭐️⭐️⭐️⭐️ (High) | RegEx `/[%_\\]/g` | Neutralizes SQL wildcard injection in dynamic search queries |
| **RFC 5987 Filename Encoder** | Backend Micro-Helper | ⭐️⭐️⭐️⭐️ (High) | URI encoding + character escaping | Handles Unicode/CJK filenames across legacy & modern browsers |
| **Safe Recovery Code Formatter** | Backend Micro-Helper | ⭐️⭐️⭐️⭐️ (High) | `crypto.randomBytes`, normalization | Generates formatted `XXXX-XXXX-XXXX-XXXX`, strips symbols before check |
| **ISO-8601 Date Formatter (`dates.ts`)** | Backend Micro-Helper | ⭐️⭐️⭐️⭐️ (High) | `Intl.DateTimeFormat` (en-CA UTC) | Fast, timezone-safe `YYYY-MM-DD` parsing and rendering |
| **Native Excel Exporter (`xlsx` / SheetJS)** | Frontend Admin Service | ⭐️⭐️⭐️⭐️⭐️ (High) | SheetJS (`xlsx`), Blob URL, i18n | Multi-column selection, dynamic auto-width calculation, localized column headers & dates |
| **Universal Database Backup & Restore Engine** | Backend & Admin Service | ⭐️⭐️⭐️⭐️⭐️ (High) | Drizzle Transactions, Node JSON Stream | Zero external binary dependency (`pg_dump`/`pg_restore`), cross-platform portability |
| **Two-Factor Password & Keyword Confirmation Modal** | Security / Admin UI Widget | ⭐️⭐️⭐️⭐️⭐️ (High) | Argon2id Re-auth, Controlled Inputs | Re-authentication before destructive ops, keyword lock ("yes"), full i18n |
| **Responsive Data Table Pagination Control** | Core Admin UI Widget | ⭐️⭐️⭐️⭐️ (High) | React State, TailwindCSS, Lucide | Per-page size selector, localized range summaries, bounded page jumps |

---

## 2. Deep-Dive: Security & Authentication Architecture

### 2.1. Enterprise Two-Factor Authentication (2FA) Subsystem
- **Backend Files:** [`backend/src/services/two-factor.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/two-factor.service.ts), [`backend/src/lib/totp.ts`](file:///Users/billylam/ai/mds/backend/src/lib/totp.ts), [`backend/src/lib/recovery-codes.ts`](file:///Users/billylam/ai/mds/backend/src/lib/recovery-codes.ts)
- **Frontend Pages/Components:** [`frontend/src/pages/admin/TwoFactorSettings.tsx`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/TwoFactorSettings.tsx), [`frontend/src/components/LoginForm.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/LoginForm.tsx)

#### Key Architectural Capabilities
1. **Signed Pre-Auth Token Handshake:**
   - Instead of storing intermediate login state in insecure cookies or memory, a cryptographically signed HMAC token (`generatePreAuthToken(userId, passwordHash)`) binds Step 1 (password verification) with Step 2 (TOTP prompt).
   - Valid for 5 minutes; verifies both expiration and timing-safe signature.
2. **Replay Attack Defense:**
   - Used OTP codes are hashed via SHA-256 and recorded in `used_totp_codes`.
   - Subsequent attempts with the same token within the drift window (30–60s) are immediately rejected with `401 REPLAY_CODE`.
   - Automated cleanup sweeps entries older than 2 minutes.
3. **Emergency Recovery Code Architecture:**
   - 10 single-use recovery codes generated per user in `XXXX-XXXX-XXXX-XXXX` format.
   - Stored in database as Argon2id hashes (`recovery_codes` table).
   - Deleted transactionally immediately upon successful consumption.
4. **Adaptive Rate-Limiting / Lockout on 2FA:**
   - 5 failed 2FA attempts within 15 minutes trigger a hard `429 TOO_MANY_ATTEMPTS`.

```typescript
// Reusable Concept: HMAC-based Stateless Pre-Auth Token
export function generatePreAuthToken(userId: number, passwordHash: string): string {
  const expiresAt = Date.now() + 5 * 60 * 1000;
  const payload = `${userId}:${expiresAt}:${passwordHash.slice(0, 16)}`;
  const signature = crypto
    .createHmac("sha256", config.COOKIE_SECRET)
    .update(payload)
    .digest("base64url");
  return `${userId}.${expiresAt}.${signature}`;
}
```

---

### 2.2. Password Hashing & OWASP Compliance
- **Backend File:** [`backend/src/lib/argon2.ts`](file:///Users/billylam/ai/mds/backend/src/lib/argon2.ts)

#### Design Highlights
- Implements **OWASP 2026 Recommended Parameters** for Argon2id:
  - `memoryCost`: 19,456 KiB (~19 MiB)
  - `timeCost`: 2 iterations
  - `parallelism`: 1 lane
- Includes a dedicated `needsRehash(hash)` helper allowing transparent, zero-downtime hash upgrades upon user login when security parameters are elevated.

---

### 2.3. Session Management & Dual-Constraint Brute-Force Defense
- **Backend Files:** [`backend/src/lib/sessions.ts`](file:///Users/billylam/ai/mds/backend/src/lib/sessions.ts), [`backend/src/middleware/authenticate.ts`](file:///Users/billylam/ai/mds/backend/src/middleware/authenticate.ts)

#### Design Highlights
- **Cryptographic Session IDs:** Uses 32-byte `crypto.randomBytes` formatted as URL-safe base64.
- **Sliding Session Expiration (`touchSession`):**
  - Session expiration is automatically extended in the background upon authenticated requests without blocking the primary request path.
- **Strict Cookie Hardening:** `HttpOnly: true`, `SameSite: "lax"`, `Secure` conditional on HTTPS.
- **Dual-Constraint Brute-Force Defense:**
  - Fastify IP-level rate limiting (e.g. 5 req / 10s).
  - DB audit-backed username lockout (e.g., 5 consecutive failed logins lock the username for 15 minutes, neutralizing distributed botnets).

---

## 3. Data Integrity & Audit Infrastructure

### 3.1. Transactional Audit Log & Difference Engine
- **Backend File:** [`backend/src/lib/audit.ts`](file:///Users/billylam/ai/mds/backend/src/lib/audit.ts)
- **Frontend File:** [`frontend/src/pages/admin/AuditLog.tsx`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/AuditLog.tsx)

#### Reusable Value
- **`diffKeys(before, after)`**: Generic deep comparison engine returning sorted array of changed property keys.
- **`writeAuditLog(tx, entry)`**: Supports execution within an existing Drizzle/Postgres database transaction (`tx`), guaranteeing that domain changes and audit trails commit or roll back atomically.
- **Structured Fields:** Records actor ID, username, action, entity type/ID, `before_values` (JSONB), `after_values` (JSONB), `changed_fields` (array), IP, and UserAgent.

---

### 3.2. Optimistic Concurrency Control (OCC) & ETags
- **Backend Files:** [`backend/src/lib/etag.ts`](file:///Users/billylam/ai/mds/backend/src/lib/etag.ts), [`backend/src/services/materials.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/materials.service.ts)

#### Pattern
1. Every record maintains a numeric `version` column.
2. Read operations compute a standard weak HTTP ETag:
   ```typescript
   export function generateETag(id: number | bigint | string, version: number): string {
     const hash = crypto.createHash("sha256").update(`${id}.${version}`).digest("hex");
     return `W/"${hash}"`;
   }
   ```
3. Update operations enforce atomic updates with concurrency checks:
   ```sql
   UPDATE materials 
   SET ..., version = version + 1, updated_at = NOW() 
   WHERE id = $1 AND version = $expectedVersion 
   RETURNING *;
   ```
4. If 0 rows are returned or version mismatched, a structured `409 VERSION_CONFLICT` error is thrown with the current server version.

---

## 4. Query, Search & Pagination Patterns

### 4.1. Safe Pattern-Matching Search (SQL Injection & Wildcard Defense)
- **Backend File:** [`backend/src/services/materials.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/materials.service.ts#L107-L127)

```typescript
export function escapeLikePattern(str: string): string {
  // Escapes %, _ and backslashes to avoid wildcards in ILIKE / LIKE queries
  return str.replace(/[%_\\]/g, "\\$&");
}
```

#### Parallel Count and Fetch Pagination
```typescript
const [totalResult, rows] = await Promise.all([
  db.select({ count: sql<number>`count(*)::int` }).from(table).where(whereClause),
  db.select(...).from(table).where(whereClause).orderBy(order).limit(pageSize).offset((page - 1) * pageSize)
]);
```
- Avoids loading full tables into memory while returning clean metadata: `{ data: [...], total }`.

---

### 4.2. Role-Based Data Serializers
- **Backend Files:** [`backend/src/serializers/material.serializer.ts`](file:///Users/billylam/ai/mds/backend/src/serializers/material.serializer.ts), [`backend/src/serializers/tracking-event.serializer.ts`](file:///Users/billylam/ai/mds/backend/src/serializers/tracking-event.serializer.ts)

- Accepts a view role context (`"public"` vs `"admin"` vs `"operator"`).
- Automatically sanitizes sensitive internal fields (internal notes, versions, user details) before returning responses.

---

## 5. Storage, Files & QR Subsystems

### 5.1. Streamed Storage & Checksum Engine
- **Backend Files:** [`backend/src/services/files.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/files.service.ts), [`backend/src/lib/files.ts`](file:///Users/billylam/ai/mds/backend/src/lib/files.ts)

1. **Streaming SHA-256 + Byte Counter:** Consumes `NodeJS.ReadableStream` and pipe writes to disk while calculating hash and size simultaneously with minimal RAM overhead.
2. **Date-Partitioned Storage Hierarchy:** Organizes disk files into `{storage_root}/{yyyy}/{mm}/{uuid}.{ext}`.
3. **Safe Content-Disposition for CJK & Unicode:** Implements `RFC 5987` encoding (`filename*=UTF-8''...`) with fallback ASCII sanitized headers.

---

## 6. Granular Break-Down: Scan UI & Verification Components

### 6.1. Vertical Audit Timeline (`TrackingTimeline`)
- **File:** [`frontend/src/components/scan/TrackingTimeline.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/TrackingTimeline.tsx)
- **Why It's Reusable:**
  - Uses CSS pseudo-elements (`before:w-0.5 before:bg-slate-200`) for continuous connected vertical lines.
  - Multi-tier support: Anonymizes sensitive location/actor/notes for public users while rendering metadata for operators/admins.
  - Interactive "Stage-for-Delete" visual state with line-through styling and undo capabilities.

---

### 6.2. Tri-State Quality Result Badge (`ResultBadge`)
- **File:** [`frontend/src/components/scan/ResultBadge.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/ResultBadge.tsx)
- **Why It's Reusable:**
  - Encapsulates Pass (Emerald), Fail (Red), and Pending (Slate) state styling.
  - Accessible pill with leading status icons (`CheckCircle`, `XCircle`, `MinusCircle`).

---

### 6.3. Frozen Security Alert Notice (`FrozenNotice`)
- **File:** [`frontend/src/components/scan/FrozenNotice.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/FrozenNotice.tsx)
- **Why It's Reusable:**
  - Clean full-screen lock alert container with `role="status"` and accessible home return action.
  - Usable for any system entity placed under administrative hold/quarantine.

---

### 6.4. Barcode Monospace Serial Row (`SerialRow`)
- **File:** [`frontend/src/components/scan/SerialRow.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/scan/SerialRow.tsx)
- **Why It's Reusable:**
  - Monospace font with `break-all` and `select-all` class, enabling instant 1-click clipboard selection for barcode scanning verification.

---

## 7. Granular Break-Down: Admin Micro-Widgets & Utilities

### 7.1. Client-Side SVG to PNG Rasterizer & Exporter (`QrPanel`)
- **File:** [`frontend/src/components/admin/QrPanel.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/QrPanel.tsx)
- **Why It's Reusable:**
  - Converts vector SVG QR elements directly to a 512x512 PNG data URL via HTML5 Canvas without any backend image generation round-trip.
  - Auto-triggers file download with custom filenames.

```typescript
const downloadSvgAsPng = (svgElement: SVGSVGElement | null, filename: string) => {
  if (!svgElement) return;
  const svgData = new XMLSerializer().serializeToString(svgElement);
  const canvas = document.createElement("canvas");
  canvas.width = 512;
  canvas.height = 512;
  const ctx = canvas.getContext("2d");
  const img = new Image();
  img.onload = () => {
    if (ctx) {
      ctx.fillStyle = "white";
      ctx.fillRect(0, 0, 512, 512);
      ctx.drawImage(img, 0, 0, 512, 512);
      const pngUrl = canvas.toDataURL("image/png");
      const downloadLink = document.createElement("a");
      downloadLink.href = pngUrl;
      downloadLink.download = filename;
      downloadLink.click();
    }
  };
  img.src = "data:image/svg+xml;base64," + btoa(unescape(encodeURIComponent(svgData)));
};
```

---

### 7.2. Entity Attachment Manager & Previewer (`FileAttachmentPanel`)
- **File:** [`frontend/src/components/admin/FileAttachmentPanel.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/FileAttachmentPanel.tsx)
- **Why It's Reusable:**
  - Multi-file drag/select staging with client-side extension and 10MB size validation.
  - Built-in preview lightbox for uploaded images and PDFs.

---

### 7.3. Inline Table Field Editor (`UserDisplayNameCell`)
- **File:** [`frontend/src/components/admin/UserDisplayNameCell.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/UserDisplayNameCell.tsx)
- **Why It's Reusable:**
  - Compact inline-editing widget with hover-reveal edit button.
  - `Enter` to save, `Escape` to cancel, automatic focus, and React Query cache invalidation.

---

### 7.4. Instant Mutation Dropdown (`UserRoleSelect`)
- **File:** [`frontend/src/components/admin/UserRoleSelect.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/UserRoleSelect.tsx)
- **Why It's Reusable:**
  - Directly triggers server mutations on change with automatic disabling during request flight.

---

### 7.5. Physical Label Card & CSS Print Engine (`PrintLabelCard`)
- **File:** [`frontend/src/components/admin/PrintLabelCard.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/PrintLabelCard.tsx)
- **Why It's Reusable:**
  - Built using physical CSS millimeter dimensions (`70mm x 35mm`).
  - Utilizes `break-inside-avoid` to guarantee clean page breaks in thermal / laser printers.

---

### 7.6. Print Preview & Batch Aggregator Modal (`PrintPreviewModal`)
- **File:** [`frontend/src/components/admin/PrintPreviewModal.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/PrintPreviewModal.tsx)
- **Why It's Reusable:**
  - 2-column x 4-row (8 labels / A4 page) print preview with error tally banners and `@media print` integration.

---

### 7.7. Intelligent Axios 401 Interceptor
- **File:** [`frontend/src/api/client.ts`](file:///Users/billylam/ai/mds/frontend/src/api/client.ts)
- **Why It's Reusable:**
  - Clears Zustand auth store on 401s while avoiding infinite reload loops on public routes (`/m/*`) and login endpoints.

---

## 8. Tiny Backend Micro-Helpers

### 8.1. RFC 5987 Unicode Filename Sanitizer
- **File:** [`backend/src/lib/files.ts`](file:///Users/billylam/ai/mds/backend/src/lib/files.ts#L67-L82)
```typescript
export function encodeRFC5987ValueChars(str: string): string {
  return encodeURIComponent(str)
    .replace(/['()]/g, escape)
    .replace(/\*/g, "%2A")
    .replace(/%(?:7C|60|5E)/g, unescape);
}

export function sanitizeAsciiFilename(filename: string): string {
  return filename.replace(/[\r\n\x00-\x1f\x7f"\\]/g, "_").replace(/[^\x20-\x7e]/g, "_");
}
```

### 8.2. High-Performance Zero-Dependency UTC Date Helpers
- **File:** [`backend/src/lib/dates.ts`](file:///Users/billylam/ai/mds/backend/src/lib/dates.ts)
```typescript
const isoDateFormatter = new Intl.DateTimeFormat("en-CA", {
  year: "numeric",
  month: "2-digit",
  day: "2-digit",
  timeZone: "UTC",
});

export function formatDate(value: Date | string | null | undefined): string | null {
  if (!value) return null;
  const d = typeof value === "string" ? new Date(value) : value;
  if (isNaN(d.getTime())) return null;
  return isoDateFormatter.format(d);
}
```

### 8.3. Recovery Code Generator & Normalizer
- **File:** [`backend/src/lib/recovery-codes.ts`](file:///Users/billylam/ai/mds/backend/src/lib/recovery-codes.ts)
```typescript
export function generateRecoveryCodes(count = 10): string[] {
  const codes: string[] = [];
  for (let i = 0; i < count; i++) {
    const raw = crypto.randomBytes(8).toString("hex").toUpperCase();
    codes.push(`${raw.slice(0, 4)}-${raw.slice(4, 8)}-${raw.slice(8, 12)}-${raw.slice(12, 16)}`);
  }
  return codes;
}

export async function hashRecoveryCode(code: string): Promise<string> {
  const normalized = code.replace(/[^A-Za-z0-9]/g, "").toUpperCase();
  return hashPassword(normalized);
}
```

---

## 9. Modular Extraction Recommendations

1. **`@app/scan-primitives`**: Extract `TrackingTimeline`, `ResultBadge`, `FrozenNotice`, and `SerialRow` into a dedicated package for verification and barcode/QR scanning frontends.
2. **`@app/auth-kit`**: TOTP verification, HMAC pre-auth tokens, recovery code generation, and Argon2id rehash triggers.
3. **`@app/db-audit`**: Generic `diffKeys()`, Drizzle audit schemas, and transactional `writeAuditLog()`.
4. **`@app/print-kit`**: `PrintLabelCard`, `PrintPreviewModal`, and physical label CSS layout rules.
5. **`@app/ui-widgets`**: `UserDisplayNameCell`, `UserRoleSelect`, `QrPanel` (SVG rasterizer), `FileAttachmentPanel`, `LanguageSwitcher`, and `ConfirmDialog`.
6. **`@app/excel-export`**: Multi-criteria client-side SheetJS binary builder with column toggles and auto-width calculation.
7. **`@app/universal-backup`**: Zero-binary dependency database snapshot & transactional restore subsystem.

---

## 10. Deep-Dive: Universal Database Backup & Restore Engine

- **Backend File:** [`backend/src/services/backup.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/backup.service.ts)
- **Frontend Page:** [`frontend/src/pages/admin/BackupRestore.tsx`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/BackupRestore.tsx)

### Key Architectural Capabilities

1. **Zero External Binary Dependency (Universal Node.js Engine):**
   - Traditional database dumps relying on OS binaries (`pg_dump`, `pg_restore`) frequently fail in diverse hosting environments (e.g. Windows dev machines without Postgres bin in PATH, or containerized environments).
   - MDS utilizes a pure Node.js table exporter & importer that writes structured, compressed JSON schema dumps (`mds_json_v1`).
   - Ensures **100% portability** across Windows local development, macOS, Linux, and Zeabur cloud containers.

2. **Transactional Integrity & Foreign Key Cascade Restoration:**
   - Database restore executes within a unified database transaction (`db.transaction(async (tx) => ...)`).
   - Automatically handles table dependency order:
     - Deletion: `tracking_events` ➔ `inspections` ➔ `files` ➔ `materials`.
     - Insertion: `materials` ➔ `inspections` ➔ `tracking_events` ➔ `files`.
   - Re-applies timezone conversions (`new Date(...)`) to timestamps and safely resets PostgreSQL primary key sequences (`setval(pg_get_serial_sequence(...), max(id), max(id) IS NOT NULL)`).

---

## 11. Deep-Dive: Multi-Criteria Native Excel Export Subsystem

- **Frontend File:** [`frontend/src/pages/admin/InventoryExport.tsx`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/InventoryExport.tsx)
- **Backend Service & Route:** [`backend/src/services/materials.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/materials.service.ts), [`backend/src/routes/materials.routes.ts`](file:///Users/billylam/ai/mds/backend/src/routes/materials.routes.ts)

### Key Architectural Capabilities

1. **Native Binary `.xlsx` Generation via SheetJS:**
   - Bypasses legacy CSV limitations (such as CJK text encoding issues in Microsoft Excel, leading zero truncation in serial numbers, and formula injection).
   - Uses `XLSX.utils.book_new()`, `XLSX.utils.aoa_to_sheet()`, and `XLSX.write(wb, { bookType: "xlsx", type: "array" })` to generate binary Excel spreadsheets.

2. **Full Dynamic i18n Localization & Header Mapping:**
   - Translates all column headers, boolean values (`Passed / Failed / Pending`, `Yes / No`), and worksheet names according to the user's active locale (`en`, `zh-TW`, `zh-CN`).

3. **Intelligent Dynamic Column Width Calculation:**
   - Calculates optimal column widths in character units based on header and data length (supporting CJK double-width characters) so exported Excel files open perfectly formatted.

---

## 12. Security Pattern: Two-Factor Re-Authentication & "yes" Confirmation Modal

- **Frontend Component:** Secure Restore Modal in [`frontend/src/pages/admin/BackupRestore.tsx`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/BackupRestore.tsx)
- **Backend Route Guard:** `POST /api/v1/backups/:filename/restore` in [`backend/src/routes/backup.routes.ts`](file:///Users/billylam/ai/mds/backend/src/routes/backup.routes.ts)

### Key Security Design

```
User clicks "Restore"
       │
       ▼
┌────────────────────────────────────────────────────────┐
│  High-Security Confirmation Modal Opened               │
│  - Input 1: Admin Password (with visibility toggle)    │
│  - Input 2: Type "yes" confirmation text               │
└────────────────────────────────────────────────────────┘
       │
       │ (Submit disabled until both valid)
       ▼
┌────────────────────────────────────────────────────────┐
│  Backend Verification Pipeline:                        │
│  1. Check confirmText.toLowerCase() === "yes"          │
│  2. Fetch user's password_hash from DB                 │
│  3. Verify password via Argon2id                       │
│  4. If invalid: return 403 Forbidden                   │
│  5. If valid: Execute Transactional Database Restore   │
└────────────────────────────────────────────────────────┘
```

---

## 13. UI Pattern: Standardized Data Table Pagination Control

- **Frontend Files:** [`frontend/src/pages/admin/BackupRestore.tsx`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/BackupRestore.tsx), [`frontend/src/pages/admin/InventoryExport.tsx`](file:///Users/billylam/ai/mds/frontend/src/pages/admin/InventoryExport.tsx)

### Key Features
- **Adaptive Page Size Selection:** `5`, `10`, `20`, `50`, `100` items per page.
- **Bound-Safe Navigation:** Disables `<` (prev) on page 1 and `>` (next) on last page.
- **Localized Range Display:** Formats range indicator `Showing {from} to {to} of {total}` across all language packs.

---

## 14. Architecture: User Deletion Lifecycle & Foreign Key Safeguards

- **Backend File:** [`backend/src/services/users.service.ts`](file:///Users/billylam/ai/mds/backend/src/services/users.service.ts)
- **Frontend Component:** [`frontend/src/components/admin/UserActions.tsx`](file:///Users/billylam/ai/mds/frontend/src/components/admin/UserActions.tsx)

### 14.1. Requirements & Intent
1. **Self-Deletion Prevention:** An administrator cannot delete their own account (`ctx.actor.id === id`), returning `403 Forbidden` to prevent accidental total lockout.
2. **Transactional Disassociation (Audit Compliance):**
   - Historical audit logs created by the user must **never** be deleted when deleting an account. The system preserves the audit record and sets `audit_log.user_id = NULL` while retaining the `username` field.
   - Uploaded files uploaded by the user are preserved with `files.uploaded_by = NULL`.
3. **Session & Security Revocation:** All active sessions (`sessions`), recovery codes (`recovery_codes`), and used TOTP records (`used_totp_codes`) are cascade-deleted immediately.

### 14.2. Database Schema Constraints & Limitations on Cloud PostgreSQL (Zeabur)
1. **Schema Column Typing (`bigint` vs `bigserial`):**
   - Foreign key reference columns such as `audit_log.user_id` and `files.uploaded_by` must use `bigint` (nullable) rather than `bigserial`. `bigserial` implies an auto-incrementing sequence and non-nullable default in PostgreSQL, blocking `SET NULL` updates.
2. **Defensive Pre-Deletion Execution:**
   - In Zeabur production environments, foreign key triggers may exist with `NO ACTION`. The backend `deleteUser()` explicitly executes nullification queries within an isolated database transaction before calling `DELETE FROM users WHERE id = :id`.
3. **Audit Log Generation:**
   - Deleting a user writes a final audit log entry with `action: "delete"`, `entity_type: "user"`, capturing the before-state snapshot for security verification.

