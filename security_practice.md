# MDS Security Architecture & Best Practices Guide

This document outlines the security architecture, defensive mechanisms, and security best practices implemented across the **Material Distribution System (MDS)** codebase and deployment pipelines.

---

## 1. Database Security & SQL Injection Prevention

### 1.1 Parameterized Queries via Typed ORM
* **No Raw String Interpolation**: All database interactions utilize Drizzle ORM's strongly typed query builder and SQL template tags (`sql` tag). Query values are passed as parameterized variables (`$1, $2, ...`) directly to the PostgreSQL binary protocol driver (`postgres.js`).
* **Zero Concatenation**: User inputs (serials, usernames, IDs, filters) are never concatenated directly into SQL strings.

### 1.2 SQL Wildcard Escaping (`LIKE` / `ILIKE`)
* User search strings (`q`) passed to `ILIKE` pattern queries are sanitized to escape SQL wildcard control characters (`%`, `_`, `\`):
  ```typescript
  export function escapeLike(str: string): string {
    return str.replace(/[%_\\]/g, "\\$&");
  }
  ```
  This prevents search query manipulation and ReDoS / sequential scan resource exhaustion.

### 1.3 Least-Privilege PostgreSQL Roles
* The application runs under a dedicated, unprivileged PostgreSQL role (`mds_app`) with strictly granted `SELECT, INSERT, UPDATE, DELETE` permissions on tables.
* Database schema migrations and structural changes are restricted to the migration owner (`mds_owner`), preventing schema poisoning from runtime web application flaws.

---

## 2. Transport Layer & Web Security

### 2.1 HTTP Strict Transport Security (HSTS)
* In production deployments, TLS termination via Nginx enforces modern transport security with long-term HSTS preload headers:
  ```nginx
  add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
  ```
* Insecure HTTP connections on port 80 are permanently redirected with `301 Moved Permanently` to `https://$host$request_uri`.

### 2.2 Modern TLS Protocols & Cipher Suites
* Only `TLSv1.2` and `TLSv1.3` are enabled. Deprecated and vulnerable protocols (`SSLv3`, `TLSv1.0`, `TLSv1.1`) and weak ciphers are disabled.
* Nginx implements OCSP stapling with caching to optimize SSL handshake performance while preserving certificate revocation checking.

### 2.3 Comprehensive HTTP Security Headers
Fastify (via `@fastify/helmet`) and Nginx inject security headers on every response:
* **Content-Security-Policy (CSP)**:
  ```http
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
  ```
* **Clickjacking Protection**: `X-Frame-Options: DENY` / `frame-ancestors 'none'`.
* **MIME Sniffing Prevention**: `X-Content-Type-Options: nosniff`.
* **Referrer Policy**: `Referrer-Policy: strict-origin-when-cross-origin`.
* **Cross-Site Scripting (XSS) Protection**: React automatically escapes untrusted dynamic data in JSX templates; zero usage of `dangerouslySetInnerHTML`.

---

## 3. Authentication & Session Management

### 3.1 Argon2id Password Hashing
* Passwords are never stored in plaintext. They are hashed using **Argon2id** (the winner of the Password Hashing Competition) with production parameters:
  * Memory cost: `65536 KB` (64 MB)
  * Time iterations: `3`
  * Parallelism: `4` threads
  * Cryptographically random 16-byte salts.

### 3.2 Secure Cookie-Based Sessions
* Sessions are tracked with opaque, cryptographically random session tokens (32-byte `crypto.randomBytes`) signed using HMAC-SHA256.
* Cookies enforce hardened flags:
  * `HttpOnly`: Inaccessible via client-side JavaScript (`document.cookie`), mitigating session theft via XSS.
  * `SameSite=Lax` / `Strict`: Prevents Cross-Site Request Forgery (CSRF).
  * `Secure`: Transmitted solely over HTTPS connections.
  * `Path=/`: Restricts cookie propagation.

### 3.3 Two-Factor Authentication (2FA / TOTP) Cryptographic Binding
* **Step 1 / Step 2 Decoupling Prevention**: Login verification generates an HMAC-SHA256 signed `preAuthToken` containing `{ userId, exp, pwHashSig }`.
* The `/api/v1/auth/2fa/challenge` endpoint verifies this signature to ensure an attacker cannot attempt 2FA verification without first completing valid password authentication.
* **Replay Protection**: Verified TOTP codes within the 30-second window are recorded in the `used_totp_codes` table to reject code replay attacks.
* **Recovery Codes**: 8-character backup recovery codes are individual single-use tokens hashed with Argon2id.

### 3.4 Rate Limiting & Account Lockout
* **IP Rate Limiting**: Enforced via `@fastify/rate-limit` on public endpoints (e.g. `/api/v1/auth/login`, `/api/v1/auth/2fa/challenge`, `/api/v1/materials/scan/:serial`).
* **Progressive Account Lockout**: Consecutive failed authentication attempts track failure counters per account, locking the account for 15 minutes after 5 consecutive failures to neutralize brute-force and dictionary attacks.

---

## 4. Authorization & Role-Based Access Control (RBAC)

### 4.1 Tiered Role Matrix
* The system defines three strict authorization levels:
  1. **Admin**: Complete system control (users, system settings, file deletion, backup/restore, immutable audit trails).
  2. **Operator**: Material batch management, inspection report entry, tracking event logging, file upload.
  3. **Viewer / Public**: Read-only public verification with strict data privacy boundary enforcement.

### 4.2 Endpoint Defense-in-Depth
* All privileged backend route handlers enforce pre-handlers `authenticate` and `requireRole(...)`.
* Frontend route guards (`AdminRoute`, role checks) prevent unauthorized UI navigation, backed unconditionally by server-side verification.

### 4.3 Data Masking & Information Leakage Prevention
* When public users scan QR codes for **Frozen** (`frozen: true`) or **Non-Public** (`sample_tested_flag: false`) materials, the backend scan service masks internal quantities, delivery dates, manufacturer details, and full tracking timelines, returning sanitized responses (`FrozenNotice` / `NotPublicNotice`) to prevent metadata leakage.

---

## 5. File Management & Upload Security

### 5.1 Storage Isolation & Directory Traversal Prevention
* Uploaded files (PDF inspection certificates, inspection images) are never stored using their client-supplied filename on disk.
* Files are stored under UUIDv4 filenames inside structured date-partitioned storage:
  ```text
  .data/files/yyyy/mm/<uuid-v4>.<sanitized-ext>
  ```
* All file access is resolved through absolute paths verified within the designated `FILES_DIR` boundary.

### 5.2 MIME-Type & Extension Whitelisting
* Uploads are validated against strict whitelist maps:
  * Allowed extensions: `.pdf`, `.png`, `.jpg`, `.jpeg`
  * Allowed MIME types: `application/pdf`, `image/png`, `image/jpeg`
* Maximum upload size is enforced both at the Nginx edge (`client_max_body_size 50m;`) and application layer (10 MB per attachment).

---

## 6. Audit Trail & Non-Repudiation

### 6.1 Immutable Audit Logging
* Critical mutations (user creation, password resets, role changes, material freezes, file deletion, settings modification, 2FA lifecycle) write directly to the `audit_log` table.
* Audit records capture:
  * Timestamp (`occurred_at`)
  * Operator ID & Username
  * Action & Target Entity Type/ID
  * Before-and-After JSON change deltas (`before_values`, `after_values`, `changed_fields`)
  * Client IP Address & User Agent
* Audit trail logs are append-only; no API or UI interface supports updating or deleting audit records.

---

## 7. Pagination & Query Resource Protection

### 7.1 Hard Parameter Validation & Denial of Service Protection
* All paginated endpoints (`/api/v1/materials`, `/api/v1/users`, `/api/v1/audit-log`, `/api/v1/files`) validate query parameters using Zod schemas with strict upper bounds:
  ```typescript
  page: z.coerce.number().int().min(1).default(1),
  page_size: z.coerce.number().int().min(1).max(100).default(20)
  ```
* Passing oversized values (e.g. `page_size=1000000`) is rejected with HTTP `400 Bad Request`, preventing memory exhaustion and database denial-of-service.
