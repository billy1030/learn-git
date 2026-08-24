# MDS 備份與還原架構手冊 (Backup & Restore Architecture Manual)

> **版本 (Version):** v2.0.0  
> **適用範圍 (Scope):** 全系統備份、物料資料、物料附件、Google Drive 雲端異地同步與災害復原  
> **目標對象 (Audience):** 系統管理員、DevOps、架構師與資安工程師  

---

## 🌟 1. 架構概述 (Architecture Overview)

MDS 內建純 Node.js 驅動的**無二進位依賴 (Zero-Binary Dependency)** 多範疇備份與還原引擎。無論在 macOS、Windows、Linux 或 Zeabur 容器雲端環境中，均無需安裝外部 `pg_dump` 或 `psql` 指令，即可進行高效率的交易安全備份與無損還原。

```
┌────────────────────────────────────────────────────────────────────────┐
│                   MDS 4 大備份與還原範疇 (Scopes)                      │
├───────────────────┬────────────┬───────────────────────────────────────┤
│ 範疇 (Scope)      │ 副檔名     │ 內容說明                              │
├───────────────────┼────────────┼───────────────────────────────────────┤
│ 1. Full System    │ `.tar.gz`  │ 全系統資料庫表格 + 實體物料附件圖檔   │
│ 2. Material Record│ `.dump`    │ 物料資料庫表格、檢驗報告、物流歷程    │
│ 3. Attachment     │ `.tar.gz`  │ 實體硬碟附件檔案 + `files` 關聯索引   │
│ 4. Users & System │ `.dump`    │ 帳號、RBAC 角色、2FA 密鑰、系統設定   │
└───────────────────┴────────────┴───────────────────────────────────────┘
```

---

## 📦 2. 4 大備份範疇詳解 (Detailed Backup Scopes)

### 2.1 Full System (全系統快照)
* **副檔名**: `.tar.gz` (內建 Gzip 串流壓縮)
* **適用情境**: 系統整機遷移、重大版本升級前保護、每週/每月全量災難備份。
* **包含內容**:
  1. 所有 PostgreSQL 資料庫表格 (`users`, `sessions`, `system_settings`, `materials`, `inspections`, `tracking_events`, `files`, `audit_log`, `backup_schedules`)。
  2. 磁碟上存儲的所有實體附件二進位檔案（PDF 檢驗證明、出廠檢驗單、物料照片），由系統自動遍歷所有 Docker Volume 掛載路徑打包。

### 2.2 Material Record (物料紀錄)
* **副檔名**: `.dump` (純 SQL 結構與數據 JSON 快照)
* **適用情境**: 每日定時輕量備份、物料資料日常維護。
* **包含內容**:
  * `materials` (物料主表)
  * `inspections` (4-狀態檢驗報告)
  * `tracking_events` (物流與追蹤事件)
  * `audit_log` (稽核紀錄)

### 2.3 Material Attachment (物料附件)
* **副檔名**: `.tar.gz`
* **適用情境**: 當物料紀錄已由數據備份保護時，單獨備份所有大型二進位附件與照片。
* **包含內容**:
  * 實體磁碟目錄中存儲的原始附件檔案 (`.data/files/{YYYY}/{MM}/{uuid}.ext`)。
  * `files` 資料庫元數據表格（檔名、MIME 類型、SHA-256 哈希值、上傳者資訊）。

### 2.4 Users & System Data (使用者與系統設定)
* **副檔名**: `.dump`
* **適用情境**: 帳號與安全政策異動後的配置備份。
* **包含內容**:
  * `users` (使用者帳號與 Argon2id 密碼雜湊)
  * `two_factor_recovery_codes` (2FA 安全備用碼)
  * `system_settings` (域名、主機、QR Code 與安全性參數)
  * `backup_schedules` (定時備份任務)

---

## 🔄 3. 還原引擎與安全驗證機制 (Restore Engine & Security)

### 3.1 獨立多階段還原技術 (Independent Multi-Stage Restore)
還原程序由獨立的還原管道處理，不同範疇的快照能安全還原且互不干擾：
1. **交易安全**: 資料庫表格還原於原子交易中執行，若有任何語法或資料結構衝突，自動全部回滾 (Rollback)。
2. **實體磁碟路徑自動重建**: 解開 `.tar.gz` 壓縮包時，系統依原創立年月（如 `.data/files/2026/08/`）自動遞迴建立目錄並寫入二進位檔案。
3. **序列計數器自動重設 (`setval`)**: 資料庫還原完成後，系統自動查詢所有資料表最大 `id`，並重設 PostgreSQL Sequence，徹底避免後續新增紀錄時產生 `duplicate key` 主鍵衝突錯誤。

### 3.2 雙重身分確認防護 (Two-Factor Safety Confirmation)
為防止誤觸還原按鈕導致資料覆蓋：
1. **管理員密碼重新驗證**: 必須輸入當前登入管理員的登入密碼。
2. **輸入 `"yes"` 明確確認**: 必須手動在確認欄位鍵入 `yes` 始可送出還原請求。

---

## ☁️ 4. Google Drive 雲端異地同步 (Cloud Off-site Sync)

MDS 提供完整的 Google Drive 異地備份同步：
1. **一鍵 OAuth 2.0 連接**: 透過 Google 帳號授權，Token 與 Client 憑證持久化儲存於 PostgreSQL，無需手動編輯伺服器環境變數。
2. **無人值守自動同步 (Unattended Autonomous Background Sync)**: 系統背景定時排程引擎在凌晨觸發備份時，自動將快照上傳至 Google Drive 專屬資料夾 (`MDS-Database-Backups`)。
3. **靜默 Token 更新 (Silent Token Refresh)**: 自動使用 `refresh_token` 更新短效存取權杖，長期運行無需反覆登入授權。
4. **雲端同步保留政策 (Remote Retention Pruning)**: 本地清理過期快照時，雲端硬碟可選擇同步清理過期檔案（支援依保留天數或依保留份數）。

---

## 💻 5. RESTful API 規格參考

| HTTP 方法 | 路由端點 | 功能說明 |
|---|---|---|
| `POST` | `/api/v1/backups/create` | 建立手動備份快照 (`scope`: `full`, `materials`, `attachments`, `users_system`) |
| `GET` | `/api/v1/backups` | 取得現有所有備份快照清單與大小 |
| `GET` | `/api/v1/backups/:filename/download` | 下載 `.dump` 或 `.tar.gz` 備份檔 |
| `POST` | `/api/v1/backups/restore` | 執行快照還原 (需驗證管理員密碼與 `confirmText`) |
| `DELETE` | `/api/v1/backups/:filename` | 刪除指定快照檔案 |
| `POST` | `/api/v1/backups/gdrive/upload` | 手動同步快照至 Google Drive |
| `POST` | `/api/v1/backups/gdrive/credentials` | 儲存 Google OAuth Client ID 與 Secret |
| `DELETE` | `/api/v1/backups/gdrive/credentials` | 清除已儲存之 Google API 憑證 |
| `GET` | `/api/v1/backups/schedules` | 取得定時備份任務清單 |
| `POST` | `/api/v1/backups/schedules` | 新增定時備份排程 (每日/每週/每月) |
