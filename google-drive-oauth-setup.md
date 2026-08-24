# MDS Google Drive OAuth 2.0 Setup & Production Deployment Guide
# (Google Drive 異地備份 OAuth 2.0 設定與生產環境部署手冊)

> **Document Version:** v1.0.0  
> **Applicable Scope:** MDS Cloud Backup, Google Drive Integration, Disaster Recovery  
> **Target Audience:** System Administrators, DevOps, Security Engineers  

---

## 🌟 Overview (概述)

MDS provides an **in-app, zero-downtime Google Drive cloud sync engine** for database snapshots. Administrators can connect Google Drive via OAuth 2.0 with a single click, without modifying backend environment (`.env`) files or restarting Docker containers.

All automated scheduled backups (Daily, Weekly, Monthly) and manual uploads are synced directly to a dedicated Google Drive folder (`MDS-Database-Backups`) with automated retention policy pruning.

---

## 📋 Prerequisites (事前準備)

- A Google Cloud Platform (GCP) Account ([https://console.cloud.google.com](https://console.cloud.google.com)).
- An active GCP Project (e.g. `seventh-seeker-506307-r2`).

---

## 🛠️ Step-by-Step Configuration Guide (設定步驟)

### Step 1: Enable Google Drive API (啟用 Google Drive API)

1. Go to **Google Cloud Console** ➔ **APIs & Services (API 和服務)** ➔ **Library (程式庫)**.
2. Search for `Google Drive API`.
3. Click on **Google Drive API** and click **ENABLE (啟用)**.

---

### Step 2: Configure OAuth Consent Screen / Google Auth Platform (設定同意畫面)

1. Go to **APIs & Services** ➔ **Google Auth Platform (或 OAuth consent screen)**.
2. **User Type (使用者類型)**: Select **External (外部)** (or **Internal** if using a company Google Workspace domain).
3. Under **Branding (品牌設定)**:
   - **App Name (應用程式名稱)**: `MDS`
   - **User Support Email (使用者支援電子郵件)**: Select your admin email (e.g., `productmanager@gmail.com`).
   - **App Logo (應用程式標誌)**: **Leave Empty / Do not upload** (Uploading a logo forces manual Google verification).
   - **Developer Contact Information (開發人員聯絡資訊)**: Enter your email (`productmanager@gmail.com`).
   - **App Links (首頁/隱私權連結)**: Leave optional links empty.
   - Click **Save and Continue (儲存並繼續)**.

---

### Step 3: Add Non-Sensitive Scope (設定存取範圍)

1. Go to **Data Access (資料存取 / Scopes)** ➔ Click **Add or Remove Scopes (新增或移除範圍)**.
2. Search for and select:
   - `https://www.googleapis.com/auth/drive.file`
   *(This non-sensitive scope allows MDS to view and manage only the files and folders that MDS creates, eliminating the need for security reviews by Google).*
3. Click **Update (更新)** ➔ **Save (儲存)**.

---

### Step 4: Publish App to Production (發布至實際運作中)

> ⚠️ **Critical for Token Longevity**: If left in `Testing` mode, Google limits Refresh Tokens to **7 days**. Publishing to `Production` grants **permanent Refresh Tokens** for uninterrupted nightly automated backups.

1. Go to **Audience (目標對象)** tab.
2. Under **Publishing status (發布狀態)**, click **PUBLISH APP (發布應用程式)**.
3. Click **Confirm (確認)**.
4. Verify the status is now **實際運作中 (In Production)**.
   *(Note: MDS requires NO manual verification from Google because it uses only the `drive.file` scope and operates within internal admin team usage).*

---

### Step 5: Create OAuth 2.0 Client Credentials (建立憑證)

1. Go to **APIs & Services** ➔ **Credentials (憑證)**.
2. Click **+ CREATE CREDENTIALS (+ 建立憑證)** ➔ Select **OAuth client ID (OAuth 用戶端 ID)**.
3. **Application type (應用程式類型)**: Select **Web application (網頁應用程式)**.
4. **Name**: `MDS Backup Client`
5. **Authorized JavaScript origins (已授權的 JavaScript 來源)**:
   - Local: `http://localhost:5173`
   - Production Domain: `https://mds.yourcompany.com`
6. **Authorized redirect URIs (已授權的重新導向 URI)**:
   - Local: `http://localhost:5173/admin/backup`
   - Production: `https://mds.yourcompany.com/admin/backup`
7. Click **CREATE (建立)**.
8. Copy your **Client ID** and **Client Secret**.

---

### Step 6: Bind Credentials in MDS Dashboard (在 MDS 後台儲存憑證與備份目錄)

1. Log in to the MDS Admin Panel ➔ Navigate to **System Backup & Restore (`/admin/backup`)**.
2. Click **Schedule Backup (排程備份)**.
3. In the Google Drive card, click **API Credentials & Folder (API 憑證與目錄設定)**.
4. Paste and configure:
   - **Google Client ID**: `xxxxxx.apps.googleusercontent.com`
   - **Google Client Secret**: `GOCSPX-xxxxxx`
   - **Backup Directory / Folder Name**: e.g. `MDS-Database-Backups` (or custom name such as `Company-MDS-Backups`)
5. Click **Save Settings to Database (儲存設定至資料庫)**.
   *(Credentials and folder settings are stored securely in PostgreSQL `system_settings` table)*.

---

### Step 7: 1-Click Connect Google Drive (一鍵連接雲端硬碟)

1. Click **一鍵連接 Google Drive (Connect Google Drive)**.
2. Log in with your Google account and grant file permission.
3. You will be redirected back to MDS with the green indicator:
   `● Connected (productmanager@gmail.com)`
4. The active backup directory is displayed (with an inline edit button to update the folder at any time).
5. Ensure the **Auto-sync (自動同步)** checkbox is checked.

---

## 🔒 Security & Disaster Recovery Architecture (安全機制與災難復原)

| Feature | Description |
| :--- | :--- |
| **Unattended Autonomous Execution (無人值守排程)** | The backup scheduler runs as an independent timer process inside the Node.js backend (`backup-scheduler.service.ts`). It executes automatically at scheduled hours (e.g. 02:00) **even when no users or administrators are logged in**. |
| **Silent Token Refresh (靜默自動續期)** | The Google OAuth `refresh_token` is stored permanently in the PostgreSQL `system_settings` table. When the short-lived access token expires, the backend automatically exchanges it with Google's OAuth server in the background without user intervention. |
| **100% Dynamic Origin Resolution (動態網域解析)** | Redirect URIs are resolved dynamically via browser runtime (`window.location.origin + '/admin/backup'`), ensuring zero-configuration compatibility across Localhost, Zeabur, and Custom Enterprise Domains without hardcoded URLs. |
| **Token Encryption & Isolation** | Refresh tokens and OAuth tokens are stored in the PostgreSQL database with server-side isolation. |
| **Non-Sensitive Scope** | Restricts Google Drive access solely to MDS-generated backup snapshots. MDS cannot access or modify personal Drive files. |
| **Dedicated & Configurable Folder** | Automatically detects, provisions, or switches to any custom folder name configured by the admin (defaults to `MDS-Database-Backups`). |
| **Configurable Retention Policy** | Retains snapshots by configurable copies or days. Administrators can toggle **Prune Cloud Copies (`prune_gdrive`)** per schedule to choose whether Google Drive snapshots are pruned in sync with local retention or preserved indefinitely. |
| **Manual & Scheduled Cloud Sync** | Both unattended background schedules and immediate manual snapshots support 1-click or checkbox auto-sync to Google Drive. |
| **Zero-Binary Export** | Pure Node.js streaming export compatible across Docker containers, Linux VPS, macOS, and Windows. |

---

## ❓ Frequently Asked Questions (常見問題)

#### Q1: Will the backup job still run if no user is logged in? (若無人登入，排程備份還會執行嗎？)
**Yes, 100% autonomously.** The backup engine and Google Drive sync run entirely on the backend server (`Node.js`). Since credentials and refresh tokens are securely stored in the PostgreSQL database (not in browser cookies or sessions), scheduled jobs will reliably trigger at midnight and upload snapshots to Google Drive even when all administrators have logged out and closed their browsers.

#### Q2: Will the connection expire after 7 days?
**No.** Because the app status in Google Cloud Console is set to **In production (實際運作中)**, the Refresh Token remains valid indefinitely until explicitly disconnected.

#### Q3: Does this require Google App Verification?
**No.** Since MDS uses the restricted scope `https://www.googleapis.com/auth/drive.file` and is used internally by authorized system administrators, Google verification is not required.

#### Q4: What happens if the domain changes?
Simply go to Google Cloud Console ➔ **Credentials** ➔ Click on your OAuth Client ID ➔ Add the new domain to **Authorized redirect URIs** (e.g. `https://new-domain.com/admin/backup`). In the MDS interface, the Redirect URI box will automatically adapt to your new domain.
