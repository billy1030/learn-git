# MDS — Complete File Structure, Line Count & Program Inventory
# (物料資料系統 — 程式檔案、程式碼行數與配置清單說明書)

> **Document Version:** v1.0.1  
> **Last Updated:** August 2026  
> **Target Audience:** Full-Stack Engineers, System Architects, DevOps, and Code Auditors

---

## 📊 Codebase Metrics & Grand Totals (專案總計指標)

| Subsystem / Layer | Total Files | Total Lines of Code | Core Technologies & Highlights |
| :--- | :---: | :---: | :--- |
| **🎨 Frontend Tier** (`frontend/src/`) | **72** | **14,141** | React 19, TypeScript, TailwindCSS v4, TanStack Query, 33 Components, 3 Languages |
| **⚙️ Backend API Tier** (`backend/src/` & `test/`) | **84** | **14,206** | Node.js 22 LTS, Fastify v5, Drizzle ORM, Argon2id, 14 Routes, 13 Services, Full Integration Tests |
| **📚 Technical Documentation** (`tech-doc/` & `README`) | **15** | **3,386** | 14 Technical Specifications, Architecture Whitepapers, Schema & Security Guides |
| **🚀 DevOps, Ingress & Deploy** (`deploy/` & Root) | **35** | **7,393** | Docker Multi-Stage, Nginx Reverse Proxy, Linux Systemd Units, Automated Scripts |
| **🏆 GRAND TOTAL (全專案總計)** | **206** | **39,126** | **Enterprise-Grade Material Data & Traceability System** |

---

## 1. ⚙️ Root & Global Configuration Files (全域配置與啟動腳本)

| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`README.md`](../README.md) | **227** | Main project overview, feature summary, live routes, and documentation hub. |
| [`Dockerfile`](../Dockerfile) | **85** | Production unified multi-stage Docker container build (Node 22 + Nginx + PostgreSQL tools). |
| [`docker-compose.yml`](../docker-compose.yml) | **31** | Local multi-service orchestrator for PostgreSQL 17 database container. |
| [`docker-compose.prod.yml`](../docker-compose.prod.yml) | **46** | Production standalone container composition for enterprise VPS. |
| [`.env.example`](../.env.example) | **25** | Environment variable template with PostgreSQL URL, session secret, and rate limits. |
| [`.dockerignore`](../.dockerignore) | **6** | Ignores `.git`, `node_modules`, and local build artifacts during container builds. |
| [`.gitignore`](../.gitignore) | **23** | Git ignore patterns for dependencies, binaries, `.env`, and temporary data directories. |
| [`startup.bat`](../startup.bat) | **70** | Windows 1-click launcher (verifies Docker, migrates DB, seeds data, launches dev servers). |
| [`startup.sh`](../startup.sh) | **68** | Linux/macOS 1-click executable launcher script. |

---

## 2. 🎨 Frontend Application (`frontend/` 前端程式)

### 🛠️ Config & Root (配置與入口)
| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`frontend/package.json`](../frontend/package.json) | **38** | Frontend dependencies (React 19, Tailwind v4, TanStack Query, i18next, SheetJS). |
| [`frontend/vite.config.ts`](../frontend/vite.config.ts) | **45** | Vite 6 bundler config with dynamic Git commit SHA (`__APP_COMMIT__`) injection. |
| [`frontend/tsconfig.json`](../frontend/tsconfig.json) | **30** | TypeScript strict compilation settings and path aliases (`@/*`). |
| [`frontend/index.html`](../frontend/index.html) | **31** | HTML entry point with inline SVG loading spinner for zero-blank-screen startup. |
| [`frontend/src/main.tsx`](../frontend/src/main.tsx) | **16** | React 19 application bootstrapping and QueryClientProvider mounting. |
| [`frontend/src/App.tsx`](../frontend/src/App.tsx) | **91** | Root application router, system language initializer, and global context providers. |
| [`frontend/src/routes.tsx`](../frontend/src/routes.tsx) | **144** | React Router route definitions for public scan and admin pages. |
| [`frontend/src/index.css`](../frontend/src/index.css) | **114** | Global Tailwind CSS styling, print CSS rules, and micro-animation keyframes. |
| [`frontend/src/i18n.ts`](../frontend/src/i18n.ts) | **27** | i18next internationalization configuration with dynamic language switching. |
| [`frontend/src/vite-env.d.ts`](../frontend/src/vite-env.d.ts) | **5** | TypeScript global declarations for Vite and build constants (`__APP_VERSION__`, `__APP_COMMIT__`). |

### 📄 Pages (`frontend/src/pages/` 頁面模組)
| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`frontend/src/pages/Login.tsx`](../frontend/src/pages/Login.tsx) | **38** | Administrator & operator login page with version and commit hash signature. |
| [`frontend/src/pages/Logout.tsx`](../frontend/src/pages/Logout.tsx) | **30** | Handles session destruction and clean redirect to login screen. |
| [`frontend/src/pages/ScanPage.tsx`](../frontend/src/pages/ScanPage.tsx) | **153** | Mobile-first public QR code scan landing page (`/m/:serial`). |
| [`frontend/src/pages/Me.tsx`](../frontend/src/pages/Me.tsx) | **347** | Current authenticated user profile view with active role badges. |
| [`frontend/src/pages/admin/Materials.tsx`](../frontend/src/pages/admin/Materials.tsx) | **387** | Central material management table with fuzzy search, filters, and pagination. |
| [`frontend/src/pages/admin/MaterialDetail.tsx`](../frontend/src/pages/admin/MaterialDetail.tsx) | **274** | Full material inspection lifecycle editor, event appender, and attachment uploader. |
| [`frontend/src/pages/admin/BatchPrint.tsx`](../frontend/src/pages/admin/BatchPrint.tsx) | **325** | 2x4 sticker printable QR code batch generator with print preview modal. |
| [`frontend/src/pages/admin/InventoryExport.tsx`](../frontend/src/pages/admin/InventoryExport.tsx) | **600** | Interactive material inventory exporter generating binary `.xlsx` spreadsheets. |
| [`frontend/src/pages/admin/FileManagement.tsx`](../frontend/src/pages/admin/FileManagement.tsx) | **483** | File management with 5 storage metric cards, Materials-aligned responsive pagination, inline renaming, and deletion. |
| [`frontend/src/pages/admin/Users.tsx`](../frontend/src/pages/admin/Users.tsx) | **270** | User management panel for admin/operator/viewer roles, creation, and deactivation. |
| [`frontend/src/pages/admin/AuditLog.tsx`](../frontend/src/pages/admin/AuditLog.tsx) | **863** | Immutable audit log viewer with date filtering, retention pruning, and Excel export. |
| [`frontend/src/pages/admin/Settings.tsx`](../frontend/src/pages/admin/Settings.tsx) | **342** | System host domain, QR code URL, and default language settings panel. |
| [`frontend/src/pages/admin/SecuritySettings.tsx`](../frontend/src/pages/admin/SecuritySettings.tsx) | **811** | Interactive security tuning engine (lockouts, argon2id memory, session TTL). |
| [`frontend/src/pages/admin/TwoFactorSettings.tsx`](../frontend/src/pages/admin/TwoFactorSettings.tsx) | **288** | TOTP 2FA enrollment with QR code scanner and recovery code generation. |
| [`frontend/src/pages/admin/BackupRestore.tsx`](../frontend/src/pages/admin/BackupRestore.tsx) | **1308** | Database backup & automated scheduler manager with staggered time presets (02:00/04:00/05:00) and 2FA safety restore. |

### 🧩 Components (`frontend/src/components/` 元件庫)
| File Path | Lines | Description & Functionality |
| :--- | :---: | :--- |
| [`frontend/src/components/NavBar.tsx`](../frontend/src/components/NavBar.tsx) | **469** | **Global Responsive Navigation Bar**: Sticky top header with role-based routing links, dynamic multi-lingual selector, mobile slide-down drawer, user profile initials avatar, password change trigger, and secure sign-out. |
| [`frontend/src/components/ProtectedRoute.tsx`](../frontend/src/components/ProtectedRoute.tsx) | **54** | **Client Route Security Guard**: Checks Zustand authentication state and validates server session with React Query (`staleTime: 5m`). Restricts unauthorized routes by role (`admin`, `operator`, `viewer`) and redirects to `/login?return_to=...`. |
| [`frontend/src/components/LoginForm.tsx`](../frontend/src/components/LoginForm.tsx) | **221** | **Two-Stage Authentication Form**: React Hook Form + Zod validation handling Stage 1 (Username/Password) and Stage 2 (TOTP 6-digit challenge code) transitions with loading spinners and brute-force lockout messaging. |
| [`frontend/src/components/LanguageSwitcher.tsx`](../frontend/src/components/LanguageSwitcher.tsx) | **36** | **Live Unilingual Switcher**: Dropdown triggering instant virtual DOM i18n language transitions (繁體中文, 簡體中文, English) without page reload. |
| [`frontend/src/components/ChangePasswordModal.tsx`](../frontend/src/components/ChangePasswordModal.tsx) | **241** | **User Credential Dialog**: Secure modal with current password verification, new password complexity validation, and instant confirmation toast. |
| [`frontend/src/components/admin/MasterMaterialEditForm.tsx`](../frontend/src/components/admin/MasterMaterialEditForm.tsx) | **810** | **All-in-One Material Hub**: Multi-tab management interface integrating Material Attributes, 4-State Inspection Tests, Supply Chain Milestones, Protected File Attachments, and direct Material Record Deletion trigger with optimistic locking. |
| [`frontend/src/components/admin/FileAttachmentPanel.tsx`](../frontend/src/components/admin/FileAttachmentPanel.tsx) | **422** | **Lab Report & Attachment Manager**: Drag-and-drop file upload container with real-time SHA-256 integrity hashing, progress indicators, inline file renaming, streaming download, and deletion. |
| [`frontend/src/components/admin/UserActions.tsx`](../frontend/src/components/admin/UserActions.tsx) | **310** | **User Account Control Menu**: Action dropdown for resetting user passwords, changing RBAC roles, toggling 2FA enforcement, and deactivating accounts. |
| [`frontend/src/components/admin/MaterialCreateForm.tsx`](../frontend/src/components/admin/MaterialCreateForm.tsx) | **276** | **New Material Wizard**: Modal form validating serial format, batch quantity, manufacturer, production date, and initial visibility settings. |
| [`frontend/src/components/admin/MaterialEditForm.tsx`](../frontend/src/components/admin/MaterialEditForm.tsx) | **262** | **Material Attribute Editor**: Inline editor for updating names, batch codes, delivery dates, and public toggle with optimistic concurrency checks. |
| [`frontend/src/components/admin/InspectionForm.tsx`](../frontend/src/components/admin/InspectionForm.tsx) | **199** | **4-State Inspection Recorder**: Form enforcing test outcomes (`Passed`, `Failed`, `Pending`, `Unknown`), inspector names, test standards, and lab certificate links. |
| [`frontend/src/components/admin/UserCreateForm.tsx`](../frontend/src/components/admin/UserCreateForm.tsx) | **191** | **User Provisioning Form**: Admin dialog for creating new operator or viewer accounts with temporary password assignment and display names. |
| [`frontend/src/components/admin/MaterialMaintenanceDeleteModal.tsx`](../frontend/src/components/admin/MaterialMaintenanceDeleteModal.tsx) | **178** | **High-Security Deletion Guard**: Modal triggered via Edit View button requiring explicit serial confirmation and deletion reason before purging material records. |
| [`frontend/src/components/admin/EventAppendForm.tsx`](../frontend/src/components/admin/EventAppendForm.tsx) | **163** | **Supply Chain Event Logger**: Form to record immutable custody transfers, warehouse dispatches, and on-site acceptance events with timestamps and GPS locations. |
| [`frontend/src/components/admin/QrPanel.tsx`](../frontend/src/components/admin/QrPanel.tsx) | **162** | **QR Code Asset Card**: Interactive QR display card with 1-click PNG image export, public scan URL copy, and direct test link launcher. |
| [`frontend/src/components/admin/PrintPreviewModal.tsx`](../frontend/src/components/admin/PrintPreviewModal.tsx) | **120** | **Physical Print Preview Modal**: Renders print-ready 2x4 sticker sheets with zoom controls, paper size guidelines, and browser print trigger. |
| [`frontend/src/components/admin/ConfirmDialog.tsx`](../frontend/src/components/admin/ConfirmDialog.tsx) | **102** | **Reusable Destructive Action Modal**: Standardized confirmation dialog with danger/warning color themes, title, description, and async loading state. |
| [`frontend/src/components/admin/PrintLabelCard.tsx`](../frontend/src/components/admin/PrintLabelCard.tsx) | **92** | **Sticker Item Renderer**: Individual label card formatted with exact pixel margins for 2-column sticker sheet printers (`repeat(2, 1fr)`). |
| [`frontend/src/components/admin/UserDisplayNameCell.tsx`](../frontend/src/components/admin/UserDisplayNameCell.tsx) | **88** | **User Avatar & Name Table Cell**: Table component rendering user display names, username handles, and colored initial avatars. |
| [`frontend/src/components/admin/UserRoleSelect.tsx`](../frontend/src/components/admin/UserRoleSelect.tsx) | **34** | **RBAC Role Selector**: Clean dropdown selector for switching between `admin`, `operator`, and `viewer` permissions. |
| [`frontend/src/components/scan/MaterialCard.tsx`](../frontend/src/components/scan/MaterialCard.tsx) | **187** | **Mobile Public Material Card**: Core presentation card rendering serial number, batch metadata, manufacturer, production month, and visibility badges. |
| [`frontend/src/components/scan/TrackingTimeline.tsx`](../frontend/src/components/scan/TrackingTimeline.tsx) | **182** | **Visual Lifecycle Timeline**: Connected vertical SVG milestone tracker illustrating production, laboratory QA, logistics dispatch, and site delivery. |
| [`frontend/src/components/scan/InspectionCard.tsx`](../frontend/src/components/scan/InspectionCard.tsx) | **133** | **Public QA Summary Card**: Renders test result badges, inspection dates, testing agencies, and downloadable certificate attachments. |
| [`frontend/src/components/scan/ResultBadge.tsx`](../frontend/src/components/scan/ResultBadge.tsx) | **42** | **Status Pill Badge**: Color-coded pill component displaying `✓ Passed (Green)`, `✗ Failed (Red)`, `🕒 Pending (Amber)`, or `❔ Unknown (Slate)`. |
| [`frontend/src/components/scan/FrozenNotice.tsx`](../frontend/src/components/scan/FrozenNotice.tsx) | **36** | **Frozen Banner Alert**: High-contrast warning banner notifying viewers that a material has been administrative locked from distribution. |
| [`frontend/src/components/scan/NotPublicNotice.tsx`](../frontend/src/components/scan/NotPublicNotice.tsx) | **36** | **Restricted Visibility Notice**: Informational card displayed when a private material is scanned by an unauthorized public user. |
| [`frontend/src/components/scan/EditControlsBanner.tsx`](../frontend/src/components/scan/EditControlsBanner.tsx) | **35** | **Quick Admin Action Bar**: Floating top bar allowing logged-in staff to jump directly from public scan view into backend edit mode. |
| [`frontend/src/components/scan/BrandBar.tsx`](../frontend/src/components/scan/BrandBar.tsx) | **33** | **Public Header Bar**: Clean, branded navigation bar displayed on public mobile scans with system logo and language picker. |
| [`frontend/src/components/scan/LoginPrompt.tsx`](../frontend/src/components/scan/LoginPrompt.tsx) | **28** | **Operator Login CTA**: Callout box encouraging authenticated field personnel to log in to view protected inspection attachments. |
| [`frontend/src/components/scan/FrozenBadge.tsx`](../frontend/src/components/scan/FrozenBadge.tsx) | **27** | **Snowflake Freeze Indicator**: Compact snowflake badge (`❄️ Frozen`) displayed in material title headers. |
| [`frontend/src/components/scan/DowngradePrompt.tsx`](../frontend/src/components/scan/DowngradePrompt.tsx) | **26** | **Tier Downgrade Banner**: Informative prompt explaining why certain fields are concealed from anonymous public viewers. |
| [`frontend/src/components/scan/SerialRow.tsx`](../frontend/src/components/scan/SerialRow.tsx) | **22** | **Click-to-Copy Serial Bar**: Interactive serial number row with 1-click clipboard copy and toast confirmation. |
| [`frontend/src/components/ui/Toast.tsx`](../frontend/src/components/ui/Toast.tsx) | **38** | **Feedback Notification Popup**: Lightweight floating toast displaying success, error, and informational messages. |

### 🌐 Locales (`frontend/src/locales/` 多國語系語庫)
| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`frontend/src/locales/zh-TW/common.json`](../frontend/src/locales/zh-TW/common.json) | **787** | Complete traditional Chinese language dictionary with common action buttons. |
| [`frontend/src/locales/zh-CN/common.json`](../frontend/src/locales/zh-CN/common.json) | **787** | Complete simplified Chinese language dictionary with common action buttons. |
| [`frontend/src/locales/en/common.json`](../frontend/src/locales/en/common.json) | **788** | Complete English language dictionary with common action buttons. |

---

## 3. ⚙️ Backend Application (`backend/` 後端 API 引擎)

### 🛠️ Config & Core (配置與核心)
| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`backend/package.json`](../backend/package.json) | **45** | Backend dependencies (Fastify 5, Drizzle ORM, node-postgres, Argon2, OTPLib, Zod). |
| [`backend/tsconfig.json`](../backend/tsconfig.json) | **18** | Backend TypeScript compilation rules targeting Node.js 22 LTS. |
| [`backend/src/index.ts`](../backend/src/index.ts) | **26** | Server entrypoint binding Port 8000 on millisecond zero and launching async DB bootstrap. |
| [`backend/src/server.ts`](../backend/src/server.ts) | **98** | Fastify instance builder registering plugins, rate limiting, and routes. |
| [`backend/src/config.ts`](../backend/src/config.ts) | **41** | Central environment variable parser with strict fallback defaults. |
| [`backend/src/types.ts`](../backend/src/types.ts) | **25** | Core application types, error codes (`AppError`), and authenticated user models. |

### 🗄️ Database & Migrations (`backend/src/db/` 資料庫綱要)
| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`backend/src/db/schema.ts`](../backend/src/db/schema.ts) | **164** | Drizzle ORM table definitions for users, sessions, materials, inspections, audit log, etc. |
| [`backend/src/db/client.ts`](../backend/src/db/client.ts) | **37** | PostgreSQL connection pool manager (`pg.Pool`) with connection reuse. |
| [`backend/src/db/auto-bootstrap.ts`](../backend/src/db/auto-bootstrap.ts) | **239** | Background asynchronous schema synchronization and initial admin account seeding. |
| [`backend/src/db/seed-materials.ts`](../backend/src/db/seed-materials.ts) | **602** | Seeder creating 9 realistic demo materials with full inspection test histories. |
| [`backend/src/db/reset-and-seed.ts`](../backend/src/db/reset-and-seed.ts) | **333** | CLI utility for wiping and re-initializing the database. |

### 🛣️ HTTP Routes (`backend/src/routes/` 路由端點)
| [`backend/src/routes/backup.routes.ts`](../backend/src/routes/backup.routes.ts) | **514** | **Universal Backup & Google Drive Cloud API**: 4-scope database & attachment snapshotting (`full`, `materials`, `attachments`, `users_system`), Google Drive OAuth 2.0 1-click connect/disconnect, in-app client credential persistence in PostgreSQL, manual upload to Google Drive, automated schedule CRUD, and 2FA safety-verified restoration. |
| [`backend/src/routes/files.routes.ts`](../backend/src/routes/files.routes.ts) | **260** | **File Management & Storage Analytics API**: Streaming multipart file uploads with SHA-256 integrity verification, `/storage-summary` system disk & DB analytics, streaming file downloads, attachment metadata queries, and secure file deletion. |
| [`backend/src/routes/users.routes.ts`](../backend/src/routes/users.routes.ts) | **203** | **User Administration API**: CRUD endpoints for creating users, updating display names/roles, resetting passwords, and force-terminating active sessions. |
| [`backend/src/routes/materials.routes.ts`](../backend/src/routes/materials.routes.ts) | **200** | **Material Operations API**: Paginated material search, advanced filtering, optimistic locking updates, batch sticker printing endpoints, and Excel `.xlsx` inventory generation. |
| [`backend/src/routes/tracking-events.routes.ts`](../backend/src/routes/tracking-events.routes.ts) | **167** | **Supply Chain Tracking API**: Appending tamper-evident tracking events (production, dispatch, site receipt) and historical milestone timeline queries. |
| [`backend/src/routes/auth.routes.ts`](../backend/src/routes/auth.routes.ts) | **149** | **Authentication & Session API**: Rate-limited `/login` with sliding lockout protection, `/logout` session destruction, `/me` profile retrieval, and `/change-password` credential updates. |
| [`backend/src/routes/two-factor.routes.ts`](../backend/src/routes/two-factor.routes.ts) | **127** | **Two-Factor Authentication API**: TOTP secret enrollment, QR code generation, 6-digit challenge verification, and single-use emergency recovery code validation. |
| [`backend/src/routes/audit.routes.ts`](../backend/src/routes/audit.routes.ts) | **102** | **Immutable Audit Log API**: Querying tamper-evident system logs with entity/user filters, retention policy cleanup, and Excel `.xlsx` audit log exports. |
| [`backend/src/routes/inspections.routes.ts`](../backend/src/routes/inspections.routes.ts) | **71** | **Quality Inspection API**: Persisting 4-state lab inspection results (`Passed`, `Failed`, `Pending`, `Unknown`) with inspector signatures and test standards. |
| [`backend/src/routes/settings.routes.ts`](../backend/src/routes/settings.routes.ts) | **71** | **System Configuration API**: Updating host domain names, QR code URLs, default languages, and security policy presets with instant RAM cache invalidation. |
| [`backend/src/routes/system.routes.ts`](../backend/src/routes/system.routes.ts) | **67** | **System Telemetry API**: Real-time server diagnostics, database connection status, memory utilization, and runtime metrics. |
| [`backend/src/routes/qr.routes.ts`](../backend/src/routes/qr.routes.ts) | **47** | **Dynamic QR Streaming API**: Dynamic rendering and streaming of high-resolution PNG and SVG QR codes for physical labeling. |
| [`backend/src/routes/scan.routes.ts`](../backend/src/routes/scan.routes.ts) | **44** | **Public Scan API**: High-speed, unauthenticated material lookup (`GET /api/v1/public/materials/:serial`) with asynchronous non-blocking scan event logging. |
| [`backend/src/routes/health.routes.ts`](../backend/src/routes/health.routes.ts) | **33** | **Liveness Probe API**: Container health check endpoint (`/healthz`) returning database ping status, app version, and live Git commit SHA. |

### 🛠️ Backend Services (`backend/src/services/` 商業邏輯服務層)
| File Path | Lines | Description & Functionality |
| :--- | :---: | :--- |
| [`backend/src/services/google-drive.service.ts`](../backend/src/services/google-drive.service.ts) | **570** | **Google Drive Cloud Sync Service**: OAuth 2.0 authorization engine, database token storage, auto-creation of `MDS-Database-Backups` folder, manual and scheduled file uploading, and remote retention pruning (by days or by copies). |
| [`backend/src/services/backup.service.ts`](../backend/src/services/backup.service.ts) | **600** | **4-Scope Universal Backup & Restore Engine**: Pure Node.js transactional database snapshotting, raw file attachment Gzip bundling (`.tar.gz`), independent multi-stage restoration, and auto-increment sequence resets. |
| [`backend/src/services/backup-scheduler.service.ts`](../backend/src/services/backup-scheduler.service.ts) | **420** | **Schedule Backup Engine**: Interval timer evaluating daily/weekly/monthly cron rules across all 4 data scopes, triggering automated backups, executing smart hierarchy de-duplication, calling Google Drive off-site sync, and running dual retention pruning. |
| [`backend/src/services/users.service.ts`](../backend/src/services/users.service.ts) | **429** | **User Management Core**: User creation, Argon2id credential hashing, role assignment, active session invalidation, and deactivation safeguards. |
| [`backend/src/services/materials.service.ts`](../backend/src/services/materials.service.ts) | **412** | **Material Business Logic**: Material CRUD, fuzzy multi-field search, freeze toggling, optimistic concurrency control, and binary Excel spreadsheet compilation. |
| [`backend/src/services/files.service.ts`](../backend/src/services/files.service.ts) | **415** | **File & Storage Analytics Service**: Upload streaming, SHA-256 integrity, MIME verification, and `getStorageSummary()` computing uploads, backups, DB, and system runtime footprint. |
| [`backend/src/services/two-factor.service.ts`](../backend/src/services/two-factor.service.ts) | **314** | **TOTP 2FA Service**: RFC 6238 time-step calculation, OTP secret encryption, pre-auth challenge tokens, and single-use recovery code hashing. |
| [`backend/src/services/tracking-events.service.ts`](../backend/src/services/tracking-events.service.ts) | **253** | **Supply Chain Event Service**: Appends timestamped custody records to `tracking_events` and compiles chronological material timelines. |
| [`backend/src/services/settings.service.ts`](../backend/src/services/settings.service.ts) | **234** | **Settings & RAM Cache**: System configuration persistence with 60-second in-memory RAM caching for sub-millisecond API response times. |
| [`backend/src/services/auth.service.ts`](../backend/src/services/auth.service.ts) | **199** | **Authentication Service**: OWASP 2026 Argon2id password verification, pre-warmed constant-time dummy hashes, and 32-byte session token generation. |
| [`backend/src/services/scan.service.ts`](../backend/src/services/scan.service.ts) | **119** | **Public Scan Resolver**: Resolves material metadata with role-based field masking and delegates scan tracking to background workers. |
| [`backend/src/services/audit.service.ts`](../backend/src/services/audit.service.ts) | **113** | **Audit Log Engine**: Search query compiler, historical log retention cleanup, and Excel `.xlsx` audit sheet generator. |
| [`backend/src/services/inspections.service.ts`](../backend/src/services/inspections.service.ts) | **110** | **Inspection QA Logic**: Validates and persists 4-state quality inspection records and attaches certificate associations. |
| [`backend/src/services/qr.service.ts`](../backend/src/services/qr.service.ts) | **53** | **QR Rendering Service**: Encodes full scan URLs into printable PNG image buffers and SVG vector documents. |
| [`backend/src/services/system-health.service.ts`](../backend/src/services/system-health.service.ts) | **39** | **Health Diagnostic Service**: Performs active PostgreSQL socket pings and collects system uptime and memory statistics. |

### 🔐 Cryptography & Libs (`backend/src/lib/` 安全加密庫)
| File Path | Lines | Description & Functionality |
| :--- | :---: | :--- |
| [`backend/src/lib/sessions.ts`](../backend/src/lib/sessions.ts) | **115** | **Session & Lockout Store**: Cryptographic 32-byte session token creation, session database lookup, and sliding-window failed login counter. |
| [`backend/src/lib/files.ts`](../backend/src/lib/files.ts) | **81** | **File Helpers**: Directory layout helpers (`.data/files/YYYY/MM/`), unique UUID filename generation, and streaming SHA-256 hashing. |
| [`backend/src/lib/audit.ts`](../backend/src/lib/audit.ts) | **57** | **Audit Dispatcher**: Structured append-only audit logger capturing before/after JSON diffs, actor user IDs, IP addresses, and User-Agents. |
| [`backend/src/lib/argon2.ts`](../backend/src/lib/argon2.ts) | **30** | **Argon2id Hasher**: Military-grade password hasher configured with OWASP 2026 parameters (`19MB RAM, 2 iterations, 1 lane`). |
| [`backend/src/lib/totp.ts`](../backend/src/lib/totp.ts) | **26** | **TOTP Helper**: Generates base32 OTP secrets, otpauth URI strings, and validates 6-digit TOTP tokens with drift tolerance. |
| [`backend/src/lib/qr.ts`](../backend/src/lib/qr.ts) | **13** | **QR Core**: Low-level barcode matrix encoder using the `qrcode` library. |
| [`backend/src/lib/etag.ts`](../backend/src/lib/etag.ts) | **6** | **ETag Generator**: Fast hash generator for optimistic locking and HTTP conditional requests (`If-Match`). |

---

## 4. 📚 Centralized Technical Documentation (`tech-doc/` 技術手冊中心)

| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`tech-doc/technical_stack_review.md`](technical_stack_review.md) | **466** | Deep technical review of architecture decisions, trade-offs, and benchmarks. |
| [`tech-doc/deployment_zeabur.md`](deployment_zeabur.md) | **451** | Zeabur Cloud PaaS deployment, environment variable configuration, and domain setup. |
| [`tech-doc/SCHEMA.md`](SCHEMA.md) | **369** | Database schema definitions, column constraints, and foreign key relations. |
| [`tech-doc/deployment_local_docker.md`](deployment_local_docker.md) | **280** | Local Docker multi-service container setup and development guide. |
| [`tech-doc/architecture-whitepaper.md`](architecture-whitepaper.md) | **207** | System architecture whitepaper, async runtime model, and restaurant analogy. |
| [`tech-doc/reusable_components.md`](reusable_components.md) | **197** | Complete inventory of reusable React UI components and prop interfaces. |
| [`tech-doc/cold-start-analysis.md`](cold-start-analysis.md) | **193** | Technical post-mortem on Port 8000 blocking, 502 prevention, and admin login latency. |
| [`tech-doc/security_audit.md`](security_audit.md) | **169** | Comprehensive security audit report and vulnerability hardening verification. |
| [`tech-doc/file-structure.md`](file-structure.md) | **230** | Full source code, file layout, line counts, and configuration inventory table. |
| [`tech-doc/zeabur_postgresql.md`](zeabur_postgresql.md) | **155** | Step-by-step guide for migrating and provisioning cloud PostgreSQL on Zeabur. |
| [`tech-doc/security_practice.md`](security_practice.md) | **140** | Production security best practices, defense-in-depth guidelines, and compliance. |
| [`tech-doc/ui-techniques.md`](ui-techniques.md) | **124** | Frontend UI techniques & print CSS guide. |
| [`tech-doc/tech-stack.md`](tech-stack.md) | **105** | Complete frontend, backend, database, security, and infrastructure technology stack. |
| [`tech-doc/security-settings.md`](security-settings.md) | **94** | Detailed parameter manual for system security policies and Argon2id tuning. |

---

## 5. 🚀 Deployment & DevOps Packages (`deploy/` 維運與佈署包)

| File Path | Lines | Description |
| :--- | :---: | :--- |
| [`deploy/ZEABUR.md`](../deploy/ZEABUR.md) | **121** | Zeabur cloud platform operational instructions and architecture diagram. |
| [`deploy/RUNBOOK.md`](../deploy/RUNBOOK.md) | **92** | Production operations, database backup automation, disaster recovery, and maintenance runbook. |
| [`deploy/package/scripts/deploy-remote.sh`](../deploy/package/scripts/deploy-remote.sh) | **93** | Remote automated distribution deployment script. |
| [`deploy/package/scripts/deploy-local.sh`](../deploy/package/scripts/deploy-local.sh) | **87** | Automated 1-click Linux VPS installer and service configurator. |
| [`deploy/package/scripts/install-oracle.sh`](../deploy/package/scripts/install-oracle.sh) | **82** | Specific installer script optimized for Oracle Linux environments. |
| [`deploy/nginx/mds-prod.conf`](../deploy/nginx/mds-prod.conf) | **71** | Production Nginx reverse proxy configuration with gzip and static asset caching. |
| [`deploy/package/scripts/db-init-oracle.sh`](../deploy/package/scripts/db-init-oracle.sh) | **56** | Oracle Linux PostgreSQL database initializer and schema migration script. |
| [`deploy/package/scripts/db-init.sh`](../deploy/package/scripts/db-init.sh) | **50** | Standalone Linux database initialization and migration runner. |
| [`deploy/systemd/mds-backend.service`](../deploy/systemd/mds-backend.service) | **36** | Linux systemd service unit file for managing the backend Node.js daemon. |
