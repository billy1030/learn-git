# MDS — Full Technology Stack & Architectural Specifications
# (物料資料系統 — 技術棧與架構規格說明書)

> **Document Version:** v1.0.1  
> **Last Updated:** August 2026  
> **Target Audience:** Technical Leads, Full-Stack Engineers, DevOps Engineers, and Auditors

---

## 🏗️ Architectural Topology

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                                   MDS SYSTEM                                     │
├──────────────────────────────────────────────────────────────────────────────────┤
│  🎨 Frontend (Client SPA)   │ React 19 + TypeScript + Vite 6 + TailwindCSS v4    │
│  ⚙️ Backend (API Engine)    │ Node.js 22 (LTS) + Fastify v5 + Drizzle ORM        │
│  🗄️ Database & Storage      │ PostgreSQL 16 + Encrypted Filesystem Volumes       │
│  🔐 Security & Crypto       │ OWASP Argon2id + RFC 6238 TOTP + Immutable Audit   │
│  🚀 Ingress & Infrastructure │ Nginx + Unified Multi-Stage Docker + Zeabur / VPS  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. 🎨 Frontend Tier (前端技術架構)

| Technology / Library | Version | Category | Architectural Role & Implementation Details |
| :--- | :--- | :--- | :--- |
| **React** | `19.0.0` | Core UI Framework | Modern component-based rendering leveraging concurrent features and action transitions. |
| **TypeScript** | `5.7.3` | Type System | Strict compile-time type-safety across all components, hooks, and API models. |
| **Vite** | `6.2.0` | Bundler & Build Tool | Instant HMR development server, Rollup production bundling, and dynamic Git commit injection. |
| **TailwindCSS** | `4.0.9` | Styling Engine | Utility-first CSS architecture with zero runtime overhead and responsive design tokens. |
| **TanStack React Query** | `5.101.4` | Server State Manager | Declarative data fetching, RAM caching, optimistic UI updates, and intelligent refetch policies. |
| **React Router DOM** | `7.2.0` | Client-Side Routing | Declarative routing with `ProtectedRoute` session verification and query parameter parsing. |
| **React Hook Form** | `7.85.0` | Form Management | High-performance uncontrolled form state handling with minimal re-render cycles. |
| **Zod** | `3.25.76` | Schema Validation | Client-side validation for authentication, material CRUD, 2FA inputs, and system settings. |
| **i18next / react-i18next** | `26.3.6` / `17.0.11` | Internationalization | Dynamic runtime localization supporting **繁體中文 (zh-TW)**, **簡體中文 (zh-CN)**, and **English (en)** with browser auto-detection. |
| **Lucide React** | `1.31.0` | Iconography | Lightweight, tree-shakable clean SVG vector icons. |
| **qrcode.react** | `4.2.0` | Barcode Engine | Client-side SVG / Canvas QR Code generation for 2x4 printable stickers and mobile scan landing. |
| **SheetJS (xlsx)** | `0.18.5` | Data Processing | In-browser binary generation and export of material inventory spreadsheets (`.xlsx`). |
| **Zustand** | `5.0.15` | Global State | Minimalist reactive state store for authenticated user identity and active roles. |

---

## 2. ⚙️ Backend Tier (後端 API 引擎)

| Technology / Library | Version | Category | Architectural Role & Implementation Details |
| :--- | :--- | :--- | :--- |
| **Node.js** | `22.x LTS` | Runtime Environment | High-performance V8 engine with native ECMAScript Modules (ESM) and async workers. |
| **Fastify** | `5.2.1` | Web Framework | Ultra-low overhead HTTP engine delivering 30,000–70,000+ RPS with compiled JSON stringification. |
| **fastify-type-provider-zod**| `4.0.2` | Type Provider | End-to-end type inference binding HTTP route schemas directly to TypeScript contracts. |
| **Drizzle ORM** | `0.38.4` | Database ORM | High-speed, lightweight TypeScript ORM compiling directly to optimized SQL queries without overhead. |
| **pg (node-postgres)** | `8.13.3` | PostgreSQL Driver | Native connection pool (`max: 20`, `idleTimeoutMillis: 30000`, `keepAlive: true`) with connection reuse. |
| **@fastify/cookie** | `11.0.2` | Session Management | Secure, signed, HTTP-only, SameSite cookie transport for defense against XSS token theft. |
| **@fastify/rate-limit** | `10.2.2` | Rate Limiter | IP and endpoint level sliding-window rate limiting to prevent brute-force login attacks. |
| **@fastify/multipart** | `9.0.3` | File Streamer | Streaming multipart handler with byte size limits and SHA-256 checksum verification on upload. |
| **@fastify/cors** | `10.0.2` | Security Middleware | Cross-Origin Resource Sharing guard preventing unauthorized third-party origin access. |

---

## 3. 🔐 Security, Identity & Cryptography (安全與密碼學)

| Component | Standard / Algorithm | Technical Implementation Details |
| :--- | :--- | :--- |
| **Password Hashing** | **Argon2id** (OWASP 2026) | Parameters: `memoryCost: 19456 KiB (~19 MiB)`, `timeCost: 2`, `parallelism: 1`. Resistant to GPU cracking and side-channel attacks. |
| **Two-Factor Auth (2FA)** | **TOTP** (RFC 6238 / RFC 4226) | Time-based 6-digit one-time passwords compatible with Google Authenticator, Microsoft Authenticator, and 1Password. |
| **Constant-Time Mitigation** | Pre-warmed Dummy Hash | Computes dummy Argon2id verification for non-existent usernames to prevent timing side-channel username enumeration. |
| **Session Identification** | Cryptographic RNG (`crypto.randomBytes`) | Generates 256-bit (32-byte) URL-safe Base64 session identifiers stored in PostgreSQL and delivered via secure cookie. |
| **File Integrity** | **SHA-256** Checksumming | Computes content hashes on all uploaded inspection attachments and reports to detect data tampering or corruption. |
| **Audit Trail** | Immutable Append-Only Ledger | Tamper-evident `audit_log` table tracking all administrative logins, record mutations, and role changes with client IP and User-Agent. |

---

## 4. 🗄️ Database & Storage Tier (資料庫與儲存)

| Component | Technology | Technical Specifications & Storage Layout |
| :--- | :--- | :--- |
| **Relational Database** | **PostgreSQL 16** | ACID-compliant enterprise RDBMS utilizing Multi-Version Concurrency Control (MVCC) and B-Tree indexing. |
| **Schema Migration** | **Drizzle Kit** (`0.30.5`) | Declarative schema definitions in `schema.ts` with automated background synchronization (`auto-bootstrap.ts`). |
| **In-Memory Cache** | Fastify Native RAM Cache | 60-second TTL memory cache for `system_settings`, reducing database read operations by >95% (<1ms response). |
| **File Attachments** | Local Filesystem / Volume | Secure file storage path (`.data/files/`) retaining original filenames, MIME types, and UUID mappings. |
| **Database Backups** | `pg_dump` / `pg_restore` | Automated single-click `.sql` archive generation with timestamped storage in `.data/backups/`. |

---

## 5. 🚀 Infrastructure, Containerization & CI/CD (維運與佈署)

| Component | Technology | Implementation & Operational Flow |
| :--- | :--- | :--- |
| **Containerization** | **Docker Multi-Stage Build** | Lightweight `node:22-bookworm-slim` base image packaging Nginx, Node.js runtime, and PostgreSQL client tools. |
| **Reverse Proxy** | **Nginx** | Reverse proxies port 80 to Fastify on `127.0.0.1:8000`, serves pre-compiled SPA assets, and enforces 1-year cache headers (`expires 1y`). |
| **Process Supervision** | POSIX Shell (`wait -n`) | Dual-PID supervisor script (`entrypoint.sh`) ensuring immediate termination and log output if either Nginx or Fastify crashes. |
| **Cloud Hosting** | **Zeabur Cloud PaaS** | Zero-config Docker deployment connected directly to GitHub with automated container orchestration. |
| **Enterprise VPS** | Standalone Linux (Ubuntu / Debian / RHEL) | Deployable via automated shell scripts (`deploy/package/scripts/deploy-local.sh`). |
| **Build Stamping** | Git Revision Parsing | Automatically burns `git rev-parse --short HEAD` into the frontend bundle (`__APP_COMMIT__`) for zero-ambiguity version auditing. |

---

## 📚 Cross-Reference Documentation
* System Whitepaper & Async Runtime: [architecture-whitepaper.md](architecture-whitepaper.md)
* Database Schema & Relationships: [SCHEMA.md](SCHEMA.md)
* Cold Start & Port 8000 Optimization: [cold-start-analysis.md](cold-start-analysis.md)
* Security Policies & Hardening: [security-settings.md](security-settings.md)
* Main System Guide: [README.md](README.md)
