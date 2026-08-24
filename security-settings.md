# MDS — Security Policy & Technical Engine Guide

> **Dedicated URL:** `/admin/security-settings` *(Hidden from main navigation menu)*  
> **Target Audience:** System Administrators, DevOps Engineers, and Security Officers.

---

## 📌 Overview

The MDS system features a dedicated, zero-scroll **Security & Access Policy Setup** dashboard that allows administrators to dynamically configure authentication brute-force protections, account lockout rules, IP request rate limiting, PostgreSQL connection pool parameters, and session lifecycles.

**Key Design Highlights:**
- **Zero Downtime / Instant Effect**: Changes saved in this interface update the PostgreSQL `system_settings` table and immediately take effect in runtime memory without restarting backend containers or server processes.
- **Hidden by Design**: This page is intentionally excluded from the primary navigation bar and user dropdown menu. It is only accessible via direct URL navigation by authenticated users with the `admin` role.
- **Universal Backup Protection**: All security configurations are automatically backed up as part of the system's database backup routines (`/admin/backup`) and are restored seamlessly during snapshot recovery.
- **Dynamic Audit Synchronization**: The audit log pruning threshold in `/admin/audit` automatically links to and follows the `audit_retention_days` value defined in this policy.

---

## 🔐 Configuration Parameters Explained

The dashboard is structured into three compact tabs:

### Tab 1: Auth & Lockout (認證與防爆破)

| Parameter | Database Key | Default | Recommended Range | Description & Security Impact |
| :--- | :--- | :---: | :---: | :--- |
| **Max Failed Password Attempts** | `login_lockout_max` | `5` | `3` – `10` | The number of consecutive incorrect password attempts for a username before the account is temporarily locked. Prevents automated credential-stuffing and dictionary attacks. |
| **Lockout Window (Minutes)** | `login_lockout_window_min` | `15` | `5` – `30` min | The sliding timeframe in which failed attempts are tracked. Once an account is locked, it automatically unlocks after this period of inactivity. |
| **IP Login Rate Limit Max** | `login_rate_limit_max` | `10` | `10` – `30` | The maximum number of login requests permitted from a single remote IP address within the rate window. Protects the API from automated bot traffic. |
| **IP Rate Window (Seconds)** | `login_rate_limit_window_sec` | `60` | `30` – `300` sec | The time window (in seconds) used to calculate the IP rate limit. |

---

### Tab 2: DB Pool & Sessions (資料庫連線池與 Session)

| Parameter | Database Key | Default | Recommended Range | Description & Performance Impact |
| :--- | :--- | :---: | :---: | :--- |
| **PostgreSQL Connection Pool Max** | `db_pool_max` | `20` | `10` – `50` sockets | Maximum number of concurrent database connections allocated in the Node.js connection pool (`pg-pool`). Higher values allow more parallel queries under heavy traffic. |
| **DB Idle Timeout (Seconds)** | `db_idle_timeout_sec` | `30` | `15` – `60` sec | The duration an unused database socket remains open before being closed to conserve container RAM. Set to `60s` on cloud platforms (Zeabur) to eliminate connection handshake latency. |
| **Session Expiration (Days)** | `session_expires_days` | `30` | `7` – `60` days | Maximum duration an inactive user session remains valid in the database. When expired, the user is required to re-authenticate. |
| **Sliding Extension Frequency (Days)** | `session_sliding_days` | `1` | `1` – `7` days | Sliding window interval. Active user interaction automatically extends the session expiration date forward by `session_expires_days`. |

---

### Tab 3: Rate Limits & Storage (速率限制與檔案容量)

| Parameter | Database Key | Default | Recommended Range | Description & System Impact |
| :--- | :--- | :---: | :---: | :--- |
| **Public QR Scan Rate Limit / 5m** | `scan_rate_limit_max` | `1000` | `500` – `2500` | Maximum anonymous scan requests allowed per 5 minutes per remote IP on `/api/v1/public/materials/:serial`. Prevents automated scrapers while keeping on-site mobile scans fast. |
| **File Upload Limit (MB)** | `file_max_mb` | `10` | `5` – `25` MB | Maximum allowed file size for test report PDFs, mill certificates, and inspection images attached to material records. |
| **Audit Log Retention (Days)** | `audit_retention_days` | `180` | `90` – `365` days | Minimum retention window for historical audit trail records. The **System Audit Log** cleanup button in `/admin/audit` dynamically uses this value (e.g. *"Keep last 90 days"*). |

---

## ⚡ 1-Click Profile Presets

At the top of the interface, administrators can choose from 5 pre-calibrated profiles:

```
[ Custom (Current DB) ]  [ ⚡ Fast Startup (Best Perf) ]  [ Balanced ]  [ Strict ]  [ Relaxed ]
```

| Preset | Target Environment | Lockout Max | Lockout Window | IP Limit / Sec | DB Pool / Idle | Session | Scan Limit | Retention |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **⚡ Fast Startup (Best Perf)** | **Zeabur Dev / High Speed** | `10` | `5 min` | `30 / 60s` | `20 / 60s` | `30 days` | `2000 / 5m` | `90 days` |
| **Custom (Current DB)** | Revert to Active Database | *Current* | *Current* | *Current* | *Current* | *Current* | *Current* | *Current* |
| **Balanced (Default)** | Standard Production | `5` | `15 min` | `10 / 60s` | `20 / 30s` | `30 days` | `1000 / 5m` | `180 days` |
| **Strict** | High-Compliance Enterprise | `3` | `30 min` | `5 / 60s` | `20 / 15s` | `7 days` | `500 / 5m` | `365 days` |
| **Relaxed** | High-Volume Internal Intranet | `10` | `5 min` | `30 / 60s` | `30 / 60s` | `60 days` | `2500 / 5m` | `90 days` |

---

## 🔄 Dynamic Audit Log Synchronization

The System Audit Log interface (`/admin/audit`) automatically links with the **Audit Log Retention Policy**:

1. When you configure `audit_retention_days` to `90` in Security Settings, the cleanup button in `/admin/audit` immediately changes to:
   * **`保留最近 90 天`** / **`Keep last 90 days`**
2. Clicking the cleanup button opens a confirmation dialog prefilled with `90` days, requiring admin password authentication and typing `"yes"` to execute the permanent prune.

---

## 🛡️ Best Practices for Zeabur Cloud Deployment

For optimal startup performance on shared container hosting (e.g. Zeabur 2 vCPU / 2–4 GB RAM):

1. Navigate to `/admin/security-settings`.
2. Click **"⚡ Fast Startup (Best Perf)"**.
3. Click **"Save Changes"**.
4. **Result:**
   - Database connection idle timeout is extended to `60s` to prevent connection drops.
   - Lockout scanning query window is reduced to `5m` for indexed memory hits.
   - Public scan rate capacity is expanded to `2,000` requests per 5 minutes for lightning-fast phone QR verification.
