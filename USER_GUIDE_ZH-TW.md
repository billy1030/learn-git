# 物料資料系統 (Material Data System — MDS) 使用者操作手冊
# User Manual & Feature Guide (by Role)

> **文件版本 (Version):** v1.0.0  
> **系統核心目標 (Core Value):** 任何人使用手機掃描任意物料的 QR Code，都能在 3 秒內取得與其權限對應、且具備防竄改驗證的完整物料履歷與檢驗資訊。

---

## 📑 目錄 (Table of Contents)

1. [系統角色與權限概覽 (Roles & Permissions Matrix)](#1-系統角色與權限概覽-roles--permissions-matrix)
2. [公開訪客 / 現場合驗人員 (Anonymous / Public Scanner)](#2-公開訪客--現場合驗人員-anonymous--public-scanner)
   - 2.1 [手機掃描與即時查詢 (Mobile QR Scanning)](#21-手機掃描與即時查詢-mobile-qr-scanning)
   - 2.2 [四態檢驗結果判定 (4-State Inspection Results)](#22-四態檢驗結果判定-4-state-inspection-results)
   - 2.3 [生命週期履歷時間軸 (Lifecycle Tracking Timeline)](#23-生命週期履歷時間軸-lifecycle-tracking-timeline)
   - 2.4 [隱私與凍結保護機制 (Privacy & Freeze Safeguards)](#24-隱私與凍結保護機制-privacy--freeze-safeguards)
3. [檢視者 (Viewer)](#3-檢視者-viewer)
   - 3.1 [系統登入與個人帳戶管理 (Login & Profile Management)](#31-系統登入與個人帳戶管理-login--profile-management)
   - 3.2 [物料清單進階查詢與檢視 (Material Search & Read-Only Inspection)](#32-物料清單進階查詢與檢視-material-search--read-only-inspection)
   - 3.3 [檢驗報告與附件下載 (Protected Report & File Download)](#33-檢驗報告與附件下載-protected-report--file-download)
   - 3.4 [庫存報表匯出預覽 (Inventory Export & Preview)](#34-庫存報表匯出預覽-inventory-export--preview)
4. [操作員 / 資料登錄員 (Operator)](#4-操作員--資料登錄員-operator)
   - 4.1 [物料資料建檔 (Material Creation)](#41-物料資料建檔-material-creation)
   - 4.2 [多分頁物料維護核心 (Master Material Management Hub)](#42-多分頁物料維護核心-master-material-management-hub)
   - 4.3 [檢驗記錄登記與更新 (Inspection Recording)](#43-檢驗記錄登記與更新-inspection-recording)
   - 4.4 [履歷事件追加 (Supply Chain Event Logging)](#44-履歷事件追加-supply-chain-event-logging)
   - 4.5 [檔案與檢測報告上傳 (File & Lab Report Attachment)](#45-檔案與檢測報告上傳-file--lab-report-attachment)
   - 4.6 [單張與 2×4 標籤批次列印 (QR Code & Batch Label Printing)](#46-單張與-24-標籤批次列印-qr-code--batch-label-printing)
   - 4.7 [物料庫存 Excel 匯出 (Native Excel Inventory Export)](#47-物料庫存-excel-匯出-native-excel-inventory-export)
5. [系統管理員 (Admin)](#5-系統管理員-admin)
   - 5.1 [使用者與帳號生命週期管理 (User Management & Lifecycle)](#51-使用者與帳號生命週期管理-user-management--lifecycle)
   - 5.2 [雙因素驗證 (2FA / TOTP) 設定與重置 (Two-Factor Authentication Setup & Reset)](#52-雙因素驗證-2fa--totp-設定與重置-two-factor-authentication-setup--reset)
   - 5.3 [物料凍結與高安全刪除 (Granular Freeze & High-Security Deletion)](#53-物料凍結與高安全刪除-granular-freeze--high-security-deletion)
   - 5.4 [專用檔案管理中心 (Dedicated File Management Center)](#54-專用檔案管理中心-dedicated-file-management-center)
   - 5.5 [不可竄改審計日誌與歸檔 (Immutable Audit Log & Retention Pruning)](#55-不可竄改審計日誌與歸檔-immutable-audit-log--retention-pruning)
   - 5.6 [全系統無二進位資料庫備份與還原 (Universal Zero-Binary Backup & Restore)](#56-全系統無二進位資料庫備份與還原-universal-zero-binary-backup--restore)
   - 5.7 [系統伺服器、網域名稱與語言設定 (System Host & Language Settings)](#57-系統伺服器網域名稱與語言設定-system-host--language-settings)
   - 5.8 [動態安全防禦與執行階段參數微調 (Dynamic Security Settings Engine)](#58-動態安全防禦與執行階段參數微調-dynamic-security-settings-engine)
6. [常見問題與疑難排解 (FAQ & Troubleshooting)](#6-常見問題與疑難排解-faq--troubleshooting)

---

## 1. 系統角色與權限概覽 (Roles & Permissions Matrix)

MDS 採用嚴格的 **角色基礎存取控制 (Role-Based Access Control, RBAC)**，依據使用者身分動態劃分介面能見度與操作功能：

| 功能模組 (Module / Feature) | 公開訪客 (Anonymous) | 檢視者 (Viewer) | 操作員 (Operator) | 系統管理員 (Admin) |
| :--- | :---: | :---: | :---: | :---: |
| **手機掃描公開頁 (`/m/:serial`)** | ✅ 公開摘要 | ✅ 完整細節 | ✅ 完整細節 | ✅ 完整細節 + 管理捷徑 |
| **系統登入驗證 (Login & 2FA)** | ❌ | ✅ | ✅ | ✅ |
| **物料清單檢視與模糊搜尋 (`/admin/materials`)** | ❌ | ✅ (唯讀) | ✅ | ✅ |
| **物料建檔、屬性修改 (`Material CRUD`)** | ❌ | ❌ | ✅ | ✅ |
| **檢驗報告登記 (`Inspection Recording`)** | ❌ | ❌ | ✅ | ✅ |
| **供應鏈歷程追加 (`Event Logging`)** | ❌ | ❌ | ✅ | ✅ |
| **附件上傳、下載、重新命名** | ❌ (僅限公開附件) | ✅ (下載) | ✅ (上傳/下載/重新命名) | ✅ (完整管理/刪除) |
| **單張 QR Code / 2×4 標籤批次列印** | ❌ | ❌ | ✅ | ✅ |
| **庫存 Excel (.xlsx) 客製化匯出** | ❌ | ✅ (預覽/匯出) | ✅ (預覽/匯出) | ✅ (預覽/匯出) |
| **物料鎖定/解凍 (`Freeze / Unfreeze`)** | ❌ | ❌ | ❌ | ✅ |
| **高安全物料刪除 (密碼驗證刪除)** | ❌ | ❌ | ❌ | ✅ |
| **使用者帳號管理 (`/admin/users`)** | ❌ | ❌ | ❌ | ✅ |
| **雙因素驗證 (2FA) 強制與緊急重置** | ❌ | ❌ | ❌ | ✅ |
| **專用檔案管理中心 (`/admin/files`)** | ❌ | ❌ | ❌ | ✅ |
| **系統審計日誌檢視、匯出與清除 (`/admin/audit`)** | ❌ | ❌ | ❌ | ✅ |
| **資料庫備份與快照還原 (`/admin/backup`)** | ❌ | ❌ | ❌ | ✅ |
| **伺服器主機與語系偏好設定 (`/admin/settings`)** | ❌ | ❌ | ❌ | ✅ |
| **執行階段動態安全參數設定 (`/admin/security-settings`)** | ❌ | ❌ | ❌ | ✅ |

---

## 2. 公開訪客 / 現場合驗人員 (Anonymous / Public Scanner)

適用對象：業主稽核員、工地驗收人員、第三方檢驗單位或公眾。

### 2.1 手機掃描與即時查詢 (Mobile QR Scanning)
* **無需安裝 App**：使用 iOS 相機、Android 原生相機或通訊軟體內建之 QR Code Scanner，對準實體標籤上的 QR Code 進行掃描。
* **快速載入**：掃描後將自動開啟網頁 `/m/:serial`，系統在 1 秒內透過骨架屏 (Skeleton Loading) 呈現檢驗摘要。

### 2.2 四態檢驗結果判定 (4-State Inspection Results)
頁面頂端將以高對比色票勳章 (Hero Badge) 醒目顯示品質檢驗結果：
* 🟢 **合格 (`✓ Passed`)**：該物料已通過品質檢驗標準，符合進場與施作規範。
* 🔴 **不合格 (`✗ Failed`)**：檢驗未達標，現場應立即隔離管制。
* 🟡 **待檢測 (`🕒 Pending`)**：樣品已取樣或送驗中，尚未取得最終檢驗報告。
* ⚪ **未知 (`❔ Unknown`)**：物料尚未建立正式檢驗程序。

### 2.3 生命週期履歷時間軸 (Lifecycle Tracking Timeline)
* 透過直條式 SVG 時間軸，清晰呈現物料自出廠、實驗室送檢、物流運輸至工地進場的每個關鍵時間點 (Timestamp) 與事件類別 (Event Type)。

### 2.4 隱私與凍結保護機制 (Privacy & Freeze Safeguards)
* **未公開物料 (Not Public)**：若管理者將物料設定為「不對外公開 (`Display to Public = 關閉`)」，公開掃描頁將安全隱藏物料細節，並提示 *「此物料目前暫不對外公開」*。
* **凍結鎖定物料 (Frozen Materials)**：若物料因爭議被管理者凍結，公開頁面將呈現「🚫 物料已鎖定」安全警示橫幅 (0% 資料外洩)，防止爭議物料流入使用。

---

## 3. 檢視者 (Viewer)

適用對象：內部審核主管、品管稽核員、客戶駐廠代表（需要檢視內部完整資訊但不可修改資料者）。

### 3.1 系統登入與個人帳戶管理 (Login & Profile Management)
* **安全登入**：前往 `/login`，輸入管理員所核發的帳號 (Username) 與密碼 (Password)。
* **多語系切換 (Language Switcher)**：支援於右上角即時切換 **繁體中文 (zh-TW)**、**简体中文 (zh-CN)** 或 **English (en)**。
* **修改密碼 (Change Password)**：點擊導覽列右上角個人大頭貼，選擇「變更密碼」，輸入目前密碼與新密碼後即可完成更新。

### 3.2 物料清單進階查詢與檢視 (Material Search & Read-Only Inspection)
* **中央物料清單 (`/admin/materials`)**：
  * **多欄位模糊搜尋 (Fuzzy Search)**：支援以物料序號 (Serial No.)、生產批號 (Production Batch)、物料名稱 (Material Name)、製造商 (Manufacturer) 進行多條件即時過濾。
  * **狀態篩選器**：可快速勾選「僅顯示凍結物料 (❄️)」或「僅顯示未對外公開物料」。
* **檢視內部物料詳情**：
  * 點擊任一物料進入詳情頁，檢視完整的生產數量、進場日期、規格細節與內部備註。

### 3.3 檢驗報告與附件下載 (Protected Report & File Download)
* 於物料詳情頁中的「附件檔案 (Files)」區塊，可即時預覽出廠檢驗證明 (Mill Certificate)、實驗室 SGS/第三方報告 (Lab Report) 等受保護之 PDF 與圖檔。
* 點擊下載按鈕，系統將透過安全串流 (Streaming Download) 進行檔案傳輸。

### 3.4 庫存報表匯出預覽 (Inventory Export & Preview)
* 進入「庫存匯出 (`/admin/inventory-export`)」，可自訂篩選條件並進行線上分頁預覽，確認無誤後可匯出包含 18 個自訂欄位的 Excel (`.xlsx`) 試算表。

---

## 4. 操作員 / 資料登錄員 (Operator)

適用對象：倉儲管理人員、產線資料登錄員、現場品管人員。

### 4.1 物料資料建檔 (Material Creation)
1. 進入「物料清單 (`/admin/materials`)」，點擊右上角 **「＋ 新增物料 (Create Material)」**。
2. 填寫必要欄位：
   * **物料序號 (Serial Number)**：例如 `MAT-2026-001`（不可重複）。
   * **物料名稱 (Material Name)** 與 **製造商 (Manufacturer)**。
   * **生產批號 (Batch No.)**、**數量 (Quantity)** 與 **單位 (Unit)**。
   * **公開狀態 (Display to Public)**：勾選以允許現場人員透過手機掃描檢視。
3. 點擊「儲存 (Save)」完成建檔。

### 4.2 多分頁物料維護核心 (Master Material Management Hub)
* 進入物料編輯頁 (`/admin/materials/:serial`)，系統採用全功能一體化介面：
  * **標籤 1：基本屬性 (General Information)**：編輯規格、批號、數量與公開設定。內建樂觀鎖 (Optimistic Locking) 機制，避免多人同時覆蓋。
  * **標籤 2：品管檢驗 (Inspection QA)**：維護檢驗數據與判定結果。
  * **標籤 3：物流與履歷 (Tracking & Logistics)**：紀錄供應鏈動態事件。
  * **標籤 4：附件管理 (File Attachments)**：上傳與管理實驗室檢驗報告。

### 4.3 檢驗記錄登記與更新 (Inspection Recording)
1. 切換至「品管檢驗 (Inspection QA)」分頁。
2. 選擇檢驗狀態：**Passed (合格)**、**Failed (不合格)**、**Pending (待檢測)** 或 **Unknown (未知)**。
3. 填寫檢驗標準 (Testing Standard)、檢驗人員姓名 (Inspector)、檢驗日期及判定摘要。
4. 點擊「更新檢驗記錄 (Save Inspection)」儲存。

### 4.4 履歷事件追加 (Supply Chain Event Logging)
1. 切換至「物流與履歷 (Tracking & Logistics)」分頁。
2. 點擊「追加事件 (Append Event)」：
   * **事件類型 (Event Type)**：如 `DISPATCH` (出庫)、`IN_TRANSIT` (運輸中)、`SITE_RECEIVED` (工地簽收)、`QA_PASSED` (品管通過)。
   * **地點 (Location)** 與 **經辦人員 (Actor / Handler)**。
   * **詳細備註 (Notes)**。
3. 點擊「提交記錄」寫入不可竄改的物料履歷流。

### 4.5 檔案與檢測報告上傳 (File & Lab Report Attachment)
1. 切換至「附件管理 (File Attachments)」分頁。
2. 拖曳或點選上傳 PDF 檢驗報告、出廠證明或現場照片。
3. 系統自動於客戶端與後端計算 **SHA-256 完整性雜湊值**，確保報告檔案未經偽造。
4. 支援在介面中直接進行檔案重新命名 (Rename) 或預覽。

### 4.6 單張與 2×4 標籤批次列印 (QR Code & Batch Label Printing)
* **單張 QR Code 匯出**：於物料頁點擊「QR Code 面板」，可複製公開連結或下載高清 PNG 圖檔。
* **2×4 貼紙標籤批次列印 (`/admin/batch-print`)**：
  1. 勾選欲列印之多筆物料序號。
  2. 點擊「預覽列印標籤 (Print Preview)」。
  3. 系統自動以標準 2 欄式貼紙格式 (`repeat(2, 1fr)`) 排版，支援縮放預覽並呼叫瀏覽器進行標準 A4/標籤紙列印。

### 4.7 物料庫存 Excel 匯出 (Native Excel Inventory Export)
1. 前往「庫存匯出 (`/admin/inventory-export`)」。
2. 設定過濾區間（關鍵字、製造商、檢驗結果、公開狀態、日期區間）。
3. 勾選需要匯出的欄位（提供 18 種彈性欄位，包含序號、批號、數量、檢驗狀態、更新時間等）。
4. 點擊 **「匯出 Excel (.xlsx)」**，系統將即時產生原生 Excel 檔案並自動計算最佳欄寬。

---

## 5. 系統管理員 (Admin)

適用對象：企業系統管理員、資安主管、DevOps 維運工程師。

### 5.1 使用者與帳號生命週期管理 (User Management & Lifecycle)
* **帳號管理介面 (`/admin/users`)**：
  * **建立使用者 (Create User)**：指派使用者名稱 (Username)、顯示名稱 (Display Name)、初始密碼，並指派 `admin`、`operator` 或 `viewer` 角色。
  * **重設密碼 (Reset Password)**：管理員可為忘記密碼之同仁指派暫時性安全密碼。
  * **停用帳號 (Deactivate)**：即時停用使用者帳號，阻斷後續登入授權。
  * **生命週期保護 (User Deletion Lifecycle Safeguards)**：系統刪除使用者時，會自動將稽核日誌與檔案上傳者關聯安全解除 (`SET NULL`)，並同步撤銷所有使用中的 Session 與 2FA 密鑰。

### 5.2 雙因素驗證 (2FA / TOTP) 設定與重置 (Two-Factor Authentication Setup & Reset)
* **個人啟用 2FA (`/admin/2fa-settings`)**：
  1. 使用 Google Authenticator、Microsoft Authenticator 或 1Password 掃描螢幕上的 QR Code。
  2. 輸入手機產生的 6 位數動態認證碼 (TOTP Token) 進行綁定。
  3. 妥善保存系統產生的 **8 組一次性備援復原碼 (Recovery Codes)**。
* **密碼學 Pre-Auth Token 保護**：登入驗證採用兩階段分離機制，第一階段密碼驗證後簽發 5 分鐘有效之 HMAC Token，嚴格要求第二階段 TOTP 挑戰，徹底防堵繞過漏洞。
* **緊急管理員重置**：若同仁遺失手機且無復原碼，管理員可在 `/admin/users` 操作選單中執行「重置 2FA」，解除該帳號之動態認證鎖定。

### 5.3 物料凍結與高安全刪除 (Granular Freeze & High-Security Deletion)
* **物料凍結機制 (Material Freeze)**：
  * 當物料發生品質爭議或法規調查時，管理員可在物料頁執行「凍結 (Freeze)」並填寫凍結原因。
  * 凍結後，公開訪客掃描將只會看到警示公告（無法存取任何數據），防止受爭議物料被誤用。
* **高安全物料刪除 (High-Security Deletion Guard)**：
  * 為避免誤刪關鍵履歷，刪除物料時彈出二次防護對話框。
  * 必須輸入物料完整序號並輸入 **管理員登入密碼** 方可執行實體資料清除。

### 5.4 專用檔案管理中心 (`/admin/files`)
* 提供全系統集中式檔案管理視圖：
  * 檢視系統內所有上傳之檢驗報告、出廠證明與附件。
  * 提供即時串流下載、線上檔名修改 (Inline Rename) 與永久銷毀刪除 (Delete) 功能。

### 5.5 不可竄改審計日誌與歸檔 (Immutable Audit Log & Retention Pruning)
* **審計日誌檢視 (`/admin/audit`)**：
  * 系統自動以資料庫級別記錄所有敏感操作（包含物料新增/修改、檢驗判定、登入失敗、密碼變更、檔案刪除、備份還原等）。
  * 支援依日期範圍、操作類型 (Action)、使用者與目標資源進行過濾。
* **匯出 Excel (.xlsx)**：一鍵將審計日誌完整匯出為格式化之 Excel 報表。
* **過期日誌安全清除 (Retention Cleanup)**：
  * 提供「保留最近 180 天記錄」清理功能。
  * 執行清理時需再次驗證管理員密碼並手動輸入 `"yes"` 二次確認。

### 5.6 全系統無二進位資料庫備份與還原 (Universal Zero-Binary Backup & Restore)
* **進入路徑 (`/admin/backup`)**：
  * 採用純 Node.js 交易式架構 (Universal Engine)，無需主機安裝 `pg_dump` 或 `pg_restore` 二進位程式，在 Windows、Linux 與 Zeabur 雲端環境皆可一鍵運行。
* **建立備份快照 (Create Backup)**：
  * **完整系統快照 (Full System Snapshot)**：包含物料履歷、使用者帳號、2FA 密鑰、系統設定與完整審計日誌。
  * **僅物料資料 (Materials Only)** / **使用者與設定 (Users & Settings)**：支援模組化彈性打包。
* **安全還原 (Safe Restore)**：
  * 點擊還原時，必須輸入管理員密碼並輸入 `"yes"` 確認，防止意外覆蓋線上運行中之資料。

### 5.7 系統伺服器、網域名稱與語言設定 (`/admin/settings`)
* **公開主機與 QR Code 網址設定**：
  * 設定公開通訊協定 (`http` / `https`)、主機名稱/IP 與通訊埠 (Port)。
  * 提供 1 鍵常用預設：`Localhost (Port 8000)`、`Localhost Proxy (Port 80)`、`Staging IP (Port 80)`、`Production Domain (Port 443)`。
  * 設定後，所有批次列印與公開標籤之 QR Code 將即時依此 Base URL 產生。
* **預設多國語系政策**：
  * **管理介面預設語系 (`default_admin_lang`)**：設定登入後後台之預設語系（自動偵測、繁體中文、簡體中文、English）。
  * **公開掃描預設語系 (`default_public_lang`)**：設定現場手機掃描時之初始呈現語言。
  * **極速快取架構**：Fastify 後端具備 60 秒記憶體快取，設定讀取延遲低於 1ms。

### 5.8 動態安全防禦與執行階段參數微調 (`/admin/security-settings`)
> 💡 *此頁面為高安全獨立設定頁，可由管理員直接於網址列輸入 `/admin/security-settings` 開啟。*

* **零停機熱更新 (Zero-Downtime Live Tuning)**：修改參數後直接寫入資料庫並即時生效，無需重啟 Docker 容器。
* **可微調核心安全與效能參數**：
  * **暴力密碼破解防禦 (Brute-Force Lockout)**：連續登入失敗鎖定門檻 (預設 5 次) 與冷卻時間滑動視窗。
  * **連線速率限制 (Rate Limiting)**：API 與公開 QR 掃描之每分鐘請求上限。
  * **PostgreSQL 連線池 (Pool Size)**：最大資料庫連線數與閒置連線釋放時間。
  * **Session 與 Token 存活期**：使用者工作階段過期時間與滑動延展時間。
  * **Argon2id 密碼雜湊強度**：記憶體使用量 (Memory Cost) 與運算耗時迭代次數。
* **環境一鍵切換範本**：
  * `⚡ Fast Startup (Best Perf)`：專為 Zeabur 雲端冷啟動最佳化之極速模式。
  * `Strict`：嚴格企業資安防護模式。
  * `Balanced`：平衡標準模式。
  * `Relaxed`：本機開發測試寬鬆模式。

---

## 6. 常見問題與疑難排解 (FAQ & Troubleshooting)

### Q1: 手機掃描 QR Code 顯示空白或連線失敗？
* **檢查項目**：
  1. 請請管理員至 `/admin/settings` 確認 **「QR Code 基礎網址 (QR Base URL)」** 是否設定為手機可連線之對外 IP 或正式網域名稱（若設定為 `localhost` 或 `127.0.0.1`，外部手機將無法連線）。
  2. 確認防火牆或雲端安全群組 (Security Group) 是否已開啟對應 Port (如 Port 80 或 443)。

### Q2: 掃描物料後出現「此物料目前暫不對外公開」？
* **說明**：該物料在系統中之「公開狀態 (Display to Public)」目前設定為關閉。
* **處理方式**：若需對外開放檢視，請請操作員 (Operator) 或管理員 (Admin) 進入物料編輯頁，將公開狀態切換為開啟並儲存。

### Q3: 掃描物料時看到「物料已鎖定 (Frozen Notice)」警告？
* **說明**：該物料已被管理員進行行政凍結。
* **處理方式**：此狀態代表該批物料可能處於品質爭議或法規檢驗審核中。請洽詢品管部門或系統管理員解除凍結。

### Q4: 忘記密碼或 2FA 手機遺失無法登入？
* **一般使用者**：請聯絡系統管理員協助重設密碼或重置 2FA。
* **系統管理員**：若啟用 2FA，請使用當初儲存的 **一次性復原碼 (Recovery Codes)** 登入；如完全無法登入，請維運人員透過伺服器環境變數 `INITIAL_ADMIN_PASSWORD` 重置初始管理員帳戶。

### Q5: 匯出 Excel 時欄位顯示亂碼或樣式錯位？
* **說明**：MDS 採用原生 SheetJS 二進位 `.xlsx` 產生引擎，已內建單語系純淨轉換與自動欄寬適配演算法。請確保使用 Microsoft Excel 2016+、LibreOffice 或 Google 試算表開啟。

---
*手冊文件結束 (End of User Manual)*
