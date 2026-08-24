# Material Distribution System (MDS) - Security Audit Report

**Date:** 2026-08-18  
**Scope:** Backend API, Database Layer, Authentication & 2FA, File Management, Deployment & Configuration  
**Auditor:** Antigravity AI Security Review  

---

## 1. Executive Summary

A comprehensive, deep security assessment was performed on the MDS codebase. The system shows solid baseline security architecture (Argon2id hashing, parameterized queries via ORM, Helmet security headers, structured audit logging, UUID file storage). 

However, critical and moderate risk areas were identified that require remediation to harden the application against advanced threats, including an outdated dependency with a known SQL identifier injection advisory, overly permissive CORS policies, MIME-type spoofing vectors, and 2FA authentication state decoupling.

---

## 2. Findings Matrix

| Ref ID | Severity | Category | Vulnerability / Issue | Target File(s) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SEC-01** | **HIGH** | Dependency / SQLi | Drizzle ORM SQL Identifier Injection (`GHSA-gpj5-g38j-94v9`) | `backend/package.json` | Action Required |
| **SEC-02** | **HIGH** | Web Security / CSRF | Overly Permissive CORS Origin Reflection with Credentials | `backend/src/plugins/cors.ts` | Action Required |
| **SEC-03** | **MEDIUM** | File Security | MIME-Type Spoofing in File Upload Validation | `backend/src/lib/files.ts` | Action Required |
| **SEC-04** | **MEDIUM** | Authentication | 2FA Challenge Endpoint Missing Pre-Auth Signed Token | `backend/src/routes/two-factor.routes.ts` | Remediated ✅ |
| **SEC-05** | **MEDIUM** | Injection | Shell String Interpolation in Backup/Restore Subprocess | `backend/src/services/backup.service.ts` | Hardening Needed |
| **SEC-06** | **LOW** | Database / ReDoS | Unescaped LIKE / ILIKE Search Wildcards (`%`, `_`) | `backend/src/services/materials.service.ts` | Hardening Needed |
| **SEC-07** | **LOW** | Rate Limiting | Missing IP Rate Limit on 2FA Challenge Endpoint | `backend/src/routes/two-factor.routes.ts` | Hardening Needed |
| **SEC-08** | **MEDIUM** | Audit Integrity | Hardcoded Fallback Token & Webhook Audit Trail Injection | `backend/src/routes/system.routes.ts` | Remediated ✅ |

---

## 3. Detailed Technical Findings & Remediations

### 3.1 [SEC-01] Drizzle ORM SQL Identifier Injection (GHSA-gpj5-g38j-94v9)
- **Severity:** High (CVSS 7.5)
- **Component:** `backend/package.json` (`drizzle-orm@0.38.4`)
- **Description:** Versions of `drizzle-orm` prior to `0.45.2` are susceptible to SQL injection in scenarios where unescaped SQL identifiers are passed to query builder functions.
- **Remediation:**
  ```bash
  npm install drizzle-orm@latest drizzle-kit@latest --prefix backend
  ```

---

### 3.2 [SEC-02] Overly Permissive CORS Origin Reflection
- **Severity:** High
- **Component:** `backend/src/plugins/cors.ts`
- **Description:**
  ```typescript
  origin: (origin, cb) => {
    if (!origin) return cb(null, true);
    return cb(null, true); // Reflects ANY origin back with Access-Control-Allow-Credentials: true
  }
  ```
  Allowing any arbitrary origin while `credentials: true` enables malicious external websites visited by an authenticated user to perform authenticated API calls on their behalf.
- **Remediation:** Replace origin reflection with an explicit whitelist based on environment configurations:
  ```typescript
  const allowedOrigins = new Set([
    config.FRONTEND_URL,
    "http://localhost:5173",
    "http://localhost",
  ]);

  origin: (origin, cb) => {
    if (!origin || allowedOrigins.has(origin)) {
      return cb(null, true);
    }
    return cb(new Error("Not allowed by CORS"), false);
  }
  ```

---

### 3.3 [SEC-03] File Upload MIME-Type Spoofing Vector
- **Severity:** Medium
- **Component:** `backend/src/lib/files.ts`
- **Description:**
  ```typescript
  export function isAllowedFormat(filename: string, mimetype?: string): boolean {
    const ext = sanitizeExtension(filename);
    if (ALLOWED_EXTENSIONS.has(ext)) return true;
    if (mimetype && ALLOWED_MIME_TYPES.has(mimetype.toLowerCase())) return true;
    return false;
  }
  ```
  The logical `OR` (`||`) allows an attacker to supply a dangerous file extension (e.g. `malicious.exe`, `exploit.html`, `test.svg`) by simply providing a spoofed HTTP header `Content-Type: application/pdf`.
- **Remediation:**
  1. Enforce strict **AND** verification on both extension and declared MIME-type.
  2. Implement magic-number byte signature checks for uploaded file headers (e.g. `%PDF-` for PDFs, `\x89PNG` for PNGs).

---

### 3.4 [SEC-04] 2FA Challenge Endpoint Decoupled from Password Auth
- **Severity:** Medium
- **Component:** `backend/src/routes/two-factor.routes.ts` (`POST /api/v1/auth/2fa/challenge`), `backend/src/services/two-factor.service.ts`
- **Description:** The `/2fa/challenge` endpoint previously accepted raw `{ userId, code }` without cryptographically verifying that Step 1 (primary password verification) had succeeded.
- **Remediation & Fix Applied:**
  1. Login Step 1 generates a 5-minute, HMAC-SHA256 signed `preAuthToken` binding the `userId`, expiration timestamp, and user password hash.
  2. The `/2fa/challenge` endpoint and `verifyTwoFactorChallenge` service verify the HMAC signature and time validity before proceeding to TOTP code or recovery code evaluation.
  3. Frontend `LoginForm` updated to capture and transmit `preAuthToken` during the challenge transition.

---

### 3.5 [SEC-05] Shell Subprocess Command Construction in Backup/Restore
- **Severity:** Medium
- **Component:** `backend/src/services/backup.service.ts`
- **Description:** Backup creation and restore invoke `child_process.exec` with string templating:
  ```typescript
  const dumpCmd = `pg_dump -h "${pgHost}" -p "${pgPort}" -U "${pgUser}" -d "${pgDb}" ...`;
  await execAsync(dumpCmd, ...);
  ```
- **Remediation:** Replace shell-based `exec` with `execFile` or `spawn` passing arguments as discrete array elements:
  ```typescript
  import { execFile } from "child_process";
  import { promisify } from "util";
  const execFileAsync = promisify(execFile);

  await execFileAsync("pg_dump", [
    "-h", pgHost,
    "-p", String(pgPort),
    "-U", pgUser,
    "-d", pgDb,
    "--format=custom",
    "-f", targetPath
  ], { env: { ...process.env, PGPASSWORD: pgPassword } });
  ```

---

### 3.6 [SEC-06] Unescaped LIKE / ILIKE Search Wildcards
- **Severity:** Low
- **Component:** `backend/src/services/materials.service.ts`
- **Description:** Search inputs (`q`) are directly interpolated into `ILIKE %${q}%` without escaping SQL wildcard characters (`%`, `_`, `\`). A user submitting `q="%%%%%"` can force full table sequential scans.
- **Remediation:** Escape special SQL wildcard characters before constructing search patterns:
  ```typescript
  function escapeLikePattern(str: string): string {
    return str.replace(/[%_\\]/g, "\\$&");
  }
  ```

---

### 3.7 [SEC-07] Missing IP Rate Limit on 2FA Challenge Endpoint
- **Severity:** Low
- **Component:** `backend/src/routes/two-factor.routes.ts`
- **Description:** While user lockout exists, IP-level rate limiting is missing on the `/2fa/challenge` endpoint.
- **Remediation:** Apply Fastify rate limiting config (`max: 10, timeWindow: '1 minute'`) to the 2FA challenge route.

---

### 3.8 [SEC-08] Hardcoded Fallback Token & Webhook Audit Trail Injection
- **Severity:** Medium
- **Component:** `backend/src/routes/system.routes.ts` (`POST /api/v1/system/backup-events`), `backend/src/config.ts`
- **Description:** The system backup webhook route used `process.env.BACKUP_HOOK_TOKEN || "mds_backup_secret_token"`. In environments where `BACKUP_HOOK_TOKEN` was omitted, an attacker knowing the default token could forge backup/restic execution events directly into the immutable `audit_log`, compromising audit trail veracity and alerting integrity.
- **Remediation & Fix Applied:**
  1. Removed the hardcoded `"mds_backup_secret_token"` fallback.
  2. Enforced strict Zod schema validation in `config.ts` requiring `BACKUP_HOOK_TOKEN` in `production` environments.
  3. Added runtime guard returning `503 Service Unavailable` if the webhook is accessed without server-side token configuration.

---

## 4. Current Robust Security Controls ✅

1. **Password Security**: Strong Argon2id hashing with secure salting, memory cost, and iteration parameters.
2. **2FA Implementation**: RFC 6238 TOTP with replay attack prevention (`used_totp_codes` table within active time window) and Argon2-hashed recovery codes.
3. **Audit Trails**: Full immutability and detailed audit logs with client IP resolution (via `trustProxy`), user agent, and before/after payloads.
4. **Account Lockout**: Brute force protection per username and rate limiting per client IP.
5. **File Storage Architecture**: Randomized UUID filenames partitioned by `yyyy/mm` subdirectories, completely preventing directory traversal and local file inclusion.
6. **HTTP Security Headers**: Fastify Helmet plugin configuring Content Security Policy, `X-Frame-Options: SAMEORIGIN`, and `X-Content-Type-Options: nosniff`.
