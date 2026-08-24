# Database Schema Documentation

This document describes the PostgreSQL database schema for the MDS system as defined in `backend/src/db/schema.ts` (using Drizzle ORM).

---

## Database Architecture & Design

- **Architecture Model**: **Relational Database (RDBMS)** running on **PostgreSQL**.
- **ORM / Query Builder**: [Drizzle ORM](file:///c:/ai/mds/backend/src/db/schema.ts) with strict typed schema definitions and SQL migrations.
- **Relational Integrity**: Uses strict relational constraints, Primary Keys (`BIGSERIAL`), Unique constraints, and Foreign Keys with explicit cascading policies (`ON DELETE CASCADE`, `ON DELETE SET NULL`).
- **Use of JSON**: This is **NOT** a NoSQL document database. JSON is strictly used via PostgreSQL's native **`JSONB`** columns exclusively for audit log delta snapshots (`audit_log.before_values` and `audit_log.after_values`) and string array columns (`audit_log.changed_fields`).

---

## Entity Relationship Summary

- **`users`**
  - `1 -> N` **`sessions`** (Cascade delete)
  - `1 -> N` **`audit_log`** (Set null on delete)
  - `1 -> N` **`files`** (Uploaded by, Set null on delete)
  - `1 -> N` **`recovery_codes`** (Cascade delete)
  - `1 -> N` **`used_totp_codes`** (Cascade delete)
- **`materials`**
  - `1 -> 1` **`inspections`** (Cascade delete)
  - `1 -> N` **`tracking_events`** (Cascade delete)
- **`files`** (Polymorphic attachment to `materials` or `inspections`)
- **`audit_log`** (Polymorphic tracking for system entities)
- **`system_settings`** (Key-value global application settings)

---

## Tables

### 1. `users`
Stores user accounts, credentials, role-based access levels, and 2FA configuration.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | Unique user identifier |
| `username` | `TEXT` | NOT NULL, UNIQUE | Username for authentication |
| `display_name` | `TEXT` | Nullable | Full or preferred display name |
| `password_hash` | `TEXT` | NOT NULL | Argon2/Bcrypt hashed password |
| `role` | `TEXT` | NOT NULL (`admin` \| `operator` \| `viewer`) | System role & permissions |
| `is_active` | `BOOLEAN` | NOT NULL, Default `true` | Account active status flag |
| `totp_secret` | `TEXT` | Nullable | Encrypted TOTP secret key |
| `totp_enabled` | `BOOLEAN` | NOT NULL, Default `false` | Whether 2FA (TOTP) is activated |
| `created_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Account creation time |
| `disabled_at` | `TIMESTAMPTZ`| Nullable | Timestamp when account was deactivated |
| `last_login_at` | `TIMESTAMPTZ`| Nullable | Last successful login timestamp |

---

### 2. `sessions`
Active session storage for cookie-based authentication and 2FA challenge tracking.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `TEXT` | Primary Key | 32-byte base64url session token ID |
| `user_id` | `BIGINT` | NOT NULL, FK -> `users.id` (ON DELETE CASCADE) | Owning user |
| `created_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Session issue time |
| `last_seen_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Last activity time |
| `expires_at` | `TIMESTAMPTZ`| NOT NULL | Session expiration time |
| `ip_addr` | `INET` | Nullable | Client IP address |
| `user_agent` | `TEXT` | Nullable | Client User-Agent string |
| `totp_verified`| `BOOLEAN` | NOT NULL, Default `false` | Whether TOTP challenge has been verified |

**Indexes:**
- `sessions_user_idx` on `(user_id)`
- `sessions_expires_idx` on `(expires_at)`

---

### 3. `materials`
Core material inventory records.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | Unique material ID |
| `serial` | `TEXT` | NOT NULL, UNIQUE | Unique material serial / code |
| `production_batch` | `TEXT` | NOT NULL | Production batch code |
| `batch_quantity` | `INTEGER` | Nullable | Quantity produced in batch |
| `name` | `TEXT` | NOT NULL | Material / item name |
| `manufacturer` | `TEXT` | NOT NULL | Manufacturer company name |
| `production_month` | `DATE` | Nullable | Production month / date |
| `delivery_date` | `DATE` | Nullable | Delivery arrival date |
| `sample_tested_flag` | `BOOLEAN` | NOT NULL, Default `false` | Flag indicating whether sample was tested |
| `frozen` | `BOOLEAN` | NOT NULL, Default `false` | Freeze/lock flag preventing modification |
| `freeze_reason` | `TEXT` | Nullable | Justification for freezing |
| `version` | `INTEGER` | NOT NULL, Default `1` | Optimistic locking version number |
| `created_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Record creation timestamp |
| `updated_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Record last modification timestamp |

**Indexes:**
- `materials_serial_idx` on `(serial)`

---

### 4. `inspections`
Lab test and quality inspection reports attached 1-to-1 to materials.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | Inspection ID |
| `material_id` | `BIGINT` | NOT NULL, UNIQUE, FK -> `materials.id` (ON DELETE CASCADE) | Material reference (1:1) |
| `lab_name` | `TEXT` | NOT NULL | Testing laboratory / testing facility |
| `inspection_standard` | `TEXT` | NOT NULL | Test standard / criteria code |
| `test_item` | `TEXT` | NOT NULL | Subject test item / parameter |
| `test_condition` | `TEXT` | Nullable | Environmental or test condition details |
| `report_date` | `DATE` | Nullable | Inspection report release date |
| `sample_size` | `INTEGER` | Nullable | Number of samples tested |
| `test_result` | `TEXT` | NOT NULL (`pass` \| `fail` \| `pending` \| `unknown`) | Test result outcome |
| `created_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Creation timestamp |
| `updated_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Last modification timestamp |

---

### 5. `tracking_events`
Timeline and lifecycle events for materials.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | Event ID |
| `material_id` | `BIGINT` | NOT NULL, FK -> `materials.id` (ON DELETE CASCADE) | Referenced material |
| `event_type` | `TEXT` | NOT NULL | Event category / action name |
| `event_date` | `TIMESTAMPTZ`| NOT NULL | When the event occurred |
| `actor` | `TEXT` | Nullable | Person or subsystem triggering the event |
| `location` | `TEXT` | Nullable | Physical location or facility |
| `notes` | `TEXT` | Nullable | Event notes / description |
| `version` | `INTEGER` | NOT NULL, Default `1` | Optimistic locking version number |
| `created_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Created timestamp |
| `updated_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Last modified timestamp |

**Indexes:**
- `tracking_events_material_idx` on `(material_id, event_date)`
- `tracking_events_version_idx` on `(id, version)`

---

### 6. `files`
Uploaded file attachments associated with materials or inspection records.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | File ID |
| `uuid_name` | `TEXT` | NOT NULL, UNIQUE | Stored file system filename (UUID) |
| `original_name` | `TEXT` | NOT NULL | User's uploaded original filename |
| `mime_type` | `TEXT` | NOT NULL | Content MIME type |
| `byte_size` | `BIGINT` | NOT NULL | File size in bytes |
| `sha256` | `TEXT` | NOT NULL | SHA-256 hash checksum |
| `entity_type` | `TEXT` | NOT NULL (`material` \| `inspection`) | Associated entity type |
| `entity_id` | `BIGINT` | NOT NULL | Associated entity ID |
| `uploaded_by` | `BIGINT` | Nullable, FK -> `users.id` (ON DELETE SET NULL) | User ID who uploaded the file |
| `uploaded_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Upload timestamp |
| `scan_status` | `TEXT` | NOT NULL, Default `'clean'` | Antivirus/malware scan status |

**Indexes:**
- `files_entity_idx` on `(entity_type, entity_id)`
- `files_uploaded_by_idx` on `(uploaded_by)`

---

### 7. `audit_log`
Immutable audit trails for administrative and operational mutations.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | Audit log ID |
| `occurred_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Action timestamp |
| `user_id` | `BIGINT` | Nullable, FK -> `users.id` (ON DELETE SET NULL) | Performing user ID |
| `username` | `TEXT` | NOT NULL | Username snapshot at event time |
| `action` | `TEXT` | NOT NULL | Action name (`CREATE`, `UPDATE`, etc.) |
| `entity_type` | `TEXT` | NOT NULL | Target entity type name |
| `entity_id` | `BIGINT` | NOT NULL | Target entity ID |
| `before_values` | `JSONB` | Nullable | Snapshot of values before modification |
| `after_values` | `JSONB` | Nullable | Snapshot of values after modification |
| `changed_fields`| `TEXT[]` | Nullable | Array of field names modified |
| `ip_addr` | `INET` | Nullable | Client IP address |
| `user_agent` | `TEXT` | Nullable | Client User-Agent header |

**Indexes:**
- `audit_log_entity_idx` on `(entity_type, entity_id, occurred_at)`
- `audit_log_user_idx` on `(user_id, occurred_at)`

---

### 8. `system_settings`
System-wide global key-value configuration. Managed via `/admin/settings` (Host, QR, Multi-Lingual) and `/admin/security-settings` (Security & Engine Parameters).

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `key` | `TEXT` | Primary Key | Configuration key |
| `value` | `TEXT` | NOT NULL | Configuration value |
| `description` | `TEXT` | Nullable | Setting description |
| `updated_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Last updated timestamp |
| `updated_by` | `TEXT` | Nullable | Username of modifier |

**Managed Keys Reference:**
- **Host & QR Networking**: `public_protocol` (`http` / `https`), `public_hostname`, `public_port`, `qr_base_url`
- **Multi-Lingual Engine**: `default_admin_lang` (`auto` / `zh-TW` / `zh-CN` / `en`), `default_public_lang` (`auto` / `zh-TW` / `zh-CN` / `en`)
- **Security & Lockout**: `login_lockout_max`, `login_lockout_window_min`, `login_rate_limit_max`, `login_rate_limit_window_sec`, `session_expires_days`, `session_sliding_days`
- **Technical Engine & Storage**: `db_pool_max`, `db_idle_timeout_sec`, `file_max_mb`, `audit_retention_days`, `scan_rate_limit_max`

---

### 9. `recovery_codes`
Backup recovery codes for two-factor authentication.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | Recovery code ID |
| `user_id` | `BIGINT` | NOT NULL, FK -> `users.id` (ON DELETE CASCADE) | Owning user |
| `code_hash` | `TEXT` | NOT NULL | Secure hash of the recovery code |
| `created_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Created timestamp |

**Indexes:**
- `recovery_codes_user_idx` on `(user_id)`

---

### 10. `used_totp_codes`
Replay attack prevention table tracking used TOTP codes within time windows.

| Column | Type | Constraints / Default | Description |
|---|---|---|---|
| `id` | `BIGSERIAL` | Primary Key | Entry ID |
| `user_id` | `BIGINT` | NOT NULL, FK -> `users.id` (ON DELETE CASCADE) | Owning user |
| `code_hash` | `TEXT` | NOT NULL | Hash of consumed TOTP code |
| `used_at` | `TIMESTAMPTZ`| NOT NULL, Default `now()` | Consumption timestamp |

**Indexes:**
- `used_totp_codes_user_idx` on `(user_id, used_at)`

---

## Sample Data Examples

Below are concrete sample records illustrating how data is structured across the tables.

### Sample: `users`
```json
{
  "id": 1,
  "username": "admin",
  "display_name": "System Admin",
  "password_hash": "$argon2id$v=19$m=65536,t=3,p=4$...",
  "role": "admin",
  "is_active": true,
  "totp_secret": "encrypted_base32_secret_string",
  "totp_enabled": true,
  "created_at": "2026-08-01T08:00:00Z",
  "disabled_at": null,
  "last_login_at": "2026-08-20T09:15:00Z"
}
```

### Sample: `materials`
```json
{
  "id": 101,
  "serial": "E280382120006422033A0001",
  "production_batch": "B2026-HC-0818",
  "batch_quantity": 2500,
  "name": "高強度阻燃密目安全網 (5.0mm 加強型)",
  "manufacturer": "宏昌塑料工業股份有限公司",
  "production_month": "2026-08-01",
  "delivery_date": "2026-08-25",
  "sample_tested_flag": true,
  "frozen": false,
  "freeze_reason": null,
  "version": 1,
  "created_at": "2026-08-10T08:30:00Z",
  "updated_at": "2026-08-16T14:20:00Z"
}
```

### Sample: `inspections`
```json
{
  "id": 1,
  "material_id": 101,
  "lab_name": "SGS 台灣檢驗科技股份有限公司 (認可代碼: L1024)",
  "inspection_standard": "CNS 14253 / GB 5725-2009",
  "test_item": "網目阻燃抗燃性、拉伸破壞強度與耐候試驗",
  "test_condition": "常溫 23°C ± 2°C / 相對濕度 50% ± 5%",
  "report_date": "2026-08-16",
  "sample_size": 20,
  "test_result": "pass",
  "created_at": "2026-08-16T10:00:00Z",
  "updated_at": "2026-08-16T10:00:00Z"
}
```

### Sample: `tracking_events`
```json
[
  {
    "id": 1,
    "material_id": 101,
    "event_type": "出廠",
    "event_date": "2026-08-10T08:30:00Z",
    "actor": "陳建國 (廠長 / 品保主管)",
    "location": "桃園生產一廠 (A區倉庫，棧板區 P-04)",
    "notes": "批次產能 2500 件全數完成高密度聚乙烯 (HDPE) 網目熱定型及阻燃浸漬處理。",
    "version": 1,
    "created_at": "2026-08-10T08:30:00Z",
    "updated_at": "2026-08-10T08:30:00Z"
  },
  {
    "id": 2,
    "material_id": 101,
    "event_type": "抽樣",
    "event_date": "2026-08-12T10:15:00Z",
    "actor": "李志偉 (品管工程師)",
    "location": "SGS 委外實驗室收件處 (新北市五股園區)",
    "notes": "依照 CNS 14253 標準隨機抽樣 20 組試樣進行實驗室阻燃破壞性測試。",
    "version": 1,
    "created_at": "2026-08-12T10:15:00Z",
    "updated_at": "2026-08-12T10:15:00Z"
  }
]
```

### Sample: `files`
```json
{
  "id": 1,
  "uuid_name": "f47ac10b-58cc-4372-a567-0e02b2c3d479.pdf",
  "original_name": "SGS_Inspection_Report_B2026_HC_0818.pdf",
  "mime_type": "application/pdf",
  "byte_size": 2458920,
  "sha256": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
  "entity_type": "inspection",
  "entity_id": 1,
  "uploaded_by": 1,
  "uploaded_at": "2026-08-16T11:00:00Z",
  "scan_status": "clean"
}
```

### Sample: `audit_log`
```json
{
  "id": 501,
  "occurred_at": "2026-08-16T14:20:00Z",
  "user_id": 1,
  "username": "admin",
  "action": "UPDATE_MATERIAL",
  "entity_type": "material",
  "entity_id": 101,
  "before_values": { "sample_tested_flag": false, "version": 1 },
  "after_values": { "sample_tested_flag": true, "version": 2 },
  "changed_fields": ["sample_tested_flag", "version"],
  "ip_addr": "192.168.1.50",
  "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
}
```

### Sample: `system_settings`
```json
{
  "key": "app_title",
  "value": "MDS 物料追蹤與履歷系統",
  "description": "系統主標題名稱",
  "updated_at": "2026-08-01T00:00:00Z",
  "updated_by": "admin"
}
```

