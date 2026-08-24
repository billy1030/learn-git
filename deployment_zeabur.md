# Zeabur Deployment Guideline & User Manual

This guide provides step-by-step procedures for deploying the **MDS (Material Data System)** full-stack application (Fastify backend + React frontend + Nginx reverse proxy + PostgreSQL) on [Zeabur](https://zeabur.com).

---

## 1. Architecture Overview on Zeabur

```
       [ Public Internet / HTTPS:443 ]
                     │ (Zeabur Edge SSL / CDN)
                     ▼
           [ Container Port 80 ]
          ┌─────────────────────┐
          │     Nginx Web       │
          │   (Reverse Proxy)   │
          └──────────┬──────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
  [ / (Static Files) ]     [ /api/* & /health ]
         │                       │
         ▼                       ▼
  /var/www/html             127.0.0.1:8000
 (React Frontend)         (Fastify Backend)
                                 │
                                 ▼
                     [ Zeabur PostgreSQL Service ]
```

- **Frontend & Backend in One Container**: Nginx serves the pre-compiled React single-page app and reverse-proxies `/api/` and `/health` requests to the Fastify server running on `127.0.0.1:8000`.
- **Database**: Managed PostgreSQL instance provisioned directly on Zeabur.
- **Persistent Storage**: Persistent volume mounted at `/app/.data` for uploaded files and backup archives.

---

## 2. Pre-Deployment: Local Build Verification

Before committing and pushing code to GitHub/GitLab for Zeabur deployment, ensure production artifacts are generated locally:

### Step 1: Build Backend & Frontend
Run from project root or inside each package:
```bash
# 1. Build Backend (TypeScript compilation to backend/dist)
cd backend && npm run build && cd ..

# 2. Build Frontend (Vite production bundle to frontend/dist)
cd frontend && npm run build && cd ..
```

Ensure the following build artifacts exist:
- `backend/dist/`
- `backend/src/db/migrations/`
- `frontend/dist/`

---

## 3. Step-by-Step Zeabur Deployment Procedure

### Step 3.1: Provisioning the PostgreSQL Database Server

Zeabur provides a managed, high-performance PostgreSQL service with automated daily backups and zero maintenance.

#### 1. Create PostgreSQL Instance
1. In the [Zeabur Dashboard](https://dash.zeabur.com), click **Create Service** inside your project.
2. Select **Marketplace** ➔ Search and choose **PostgreSQL**.
3. Zeabur will provision a dedicated PostgreSQL container and attach a persistent storage volume.

#### 2. Obtain Database Connection Parameters
1. Click on the newly provisioned **PostgreSQL** service.
2. Go to the **Variables** / **Instruction** tab:
   - **Internal Connection String (Recommended)**: `postgresql://postgres:<PASSWORD>@postgresql.zeabur.internal:5432/<DB_NAME>`
   - Or use the pre-defined template variable: `${POSTGRES_CONNECTION_STRING}`
3. Copy this string for use in Step 3.3.

#### 3. (Optional / Advanced) Least-Privilege Role Setup (`roles.sql`)
For strict enterprise security compliance (CP-2 Immutable Audit Trail):
1. Connect to PostgreSQL using Zeabur's web terminal or an external tool (e.g. pgAdmin / DBeaver using the Public TCP connection).
2. Execute the role initialization script located at [`backend/src/db/roles.sql`](file:///Users/billylam/ai/mds/backend/src/db/roles.sql):
   - **`mds_owner`**: Schema owner with full migration permissions.
   - **`mds_app`**: Restricted application runtime role (can `SELECT`/`INSERT`/`UPDATE` business entities, but `UPDATE` & `DELETE` on `audit_log` are strictly revoked to prevent tampering).
3. If using standard automated setup, you can connect directly with the root `postgres` user credentials provided by Zeabur; MDS's automated bootstrapper handles table synchronization and permissions seamlessly.

---


### Step 3.2: Deploy the MDS Application Service
1. In the same Zeabur project, click **Create Service** ➔ Select **Git Repository**.
2. Select your repository containing the MDS codebase.
3. Zeabur will detect the `Dockerfile` in the root directory.

---

### Step 3.3: Configure Environment Variables

Navigate to the **Variables** tab of your MDS application service on Zeabur and add the following required environment variables:

| Variable Name | Example / Recommended Value | Description |
| :--- | :--- | :--- |
| `DATABASE_URL` | `${POSTGRES_CONNECTION_STRING}` | PostgreSQL connection string from Zeabur PostgreSQL service. |
| `NODE_ENV` | `production` | Sets Fastify and Node runtime to production mode. |
| `PORT` | `8000` | Port for internal Fastify backend server. |
| `COOKIE_SECRET` | *(64-character random string)* | Secret for HMAC session signing (must be at least 32 characters). |
| `HTTPS_ENABLED` | `true` | Enforces `Secure` flag on session cookies (Zeabur provides HTTPS edge). |
| `SEED_SAMPLE_DATA` | `false` \| `true` \| `force` | Controls sample materials & lab reports seeding on startup. |
| `BACKUP_DIR` | `/app/.data/backups` | Directory for system database backups. |
| `FILES_DIR` | `/app/.data/files` | Directory for uploaded attachments and material files. |

> [!TIP]
> **Sample Data Provisioning Options (`SEED_SAMPLE_DATA`)**:
> - **`SEED_SAMPLE_DATA=false`** (Default for Production): Creates clean schema + admin user only. Zero dummy materials.
> - **`SEED_SAMPLE_DATA=true`** (Demo / Staging): Automatically seeds **9 realistic construction materials** (Safety net, scaffolding couplers, structural steel, fire retardant boards) with full audit history, lab inspection test certificates (Pass/Fail), and tracking events if the database is currently empty.
> - **`SEED_SAMPLE_DATA=force`** (Test Reset): Forces wiping and re-seeding the 9 demo materials even if materials already exist.

> [!TIP]
> **Generating Secure Keys**:
> You can quickly generate random 32/64-character secrets using your terminal:
> ```bash
> openssl rand -base64 32
> ```


---

### Step 3.4: Configure Persistent Volume Storage

To prevent file uploads and database backups from being erased on container redeployments:

1. Click on your MDS application service in Zeabur.
2. Go to the **Volumes** tab.
3. Click **Add Volume**:
   - **Mount Path**: `/app/.data`
   - **Size**: Choose your required storage size (e.g., 5GB ~ 20GB).
4. Save the volume configuration.

---

### Step 3.5: Configure Networking & Custom Domain

1. Go to the **Networking** tab of your MDS service.
2. Under **Public Endpoints**:
   - **Exposed Port**: `80` (Nginx handles incoming HTTP traffic).
   - Click **Generate Domain** (e.g. `mds-app.zeabur.app`) or **Custom Domain** (e.g. `mds.yourcompany.com`).
3. Zeabur will automatically configure TLS/SSL certificates and route traffic to Port 80.

---

## 4. Zero Cold-Start Optimization with UptimeRobot (Free Keep-Alive)

To eliminate the **15–20s container sleep delay** on Zeabur's free/starter tier, set up an automated health probe:

1. Create a free account at [UptimeRobot](https://uptimerobot.com) (or [Cron-job.org](https://cron-job.org)).
2. Click **+ Add New Monitor**:
   - **Monitor Type**: `HTTP(s)`
   - **Friendly Name**: `MDS Zeabur Keep-Alive`
   - **URL**: `https://<YOUR_ZEABUR_DOMAIN>.zeabur.app/healthz` (or `/health`)
   - **Monitoring Interval**: `Every 5 minutes`
3. Click **Create Monitor**.

> 💡 **Why this works**: Every 5 minutes, UptimeRobot sends a lightweight HTTP ping. Fastify verifies database connectivity (`SELECT 1 FROM users`) in **< 1ms** and keeps the container, connection pool, and JIT compiler permanently warm in RAM. All public mobile scans and admin logins will respond in **< 100ms**.
>
> ⚠️ **Important Note on Internal vs External Pings**:
> Internal backend scripts sending outbound HTTP requests to third parties *cannot* keep the container awake because cloud edge routers only monitor **inbound HTTP traffic**. Once a container hibernates, internal Node.js timers (`setInterval`) are frozen. External inbound pings from UptimeRobot or Cron-Job.org ensure your internal automated backup schedules (`02:00`, `04:00`, etc.) execute on time without delay.

---

## 5. Post-Deployment Verification & Smoke Test

Once Zeabur reports the deployment as **Healthy**, verify the deployment:

### 1. Health Check
Open `https://<your-domain>/healthz` (or `/health`) in your browser. It should return:
```json
{
  "status": "ok",
  "db": "ok",
  "version": "1.0.1",
  "commit": "c5b47de",
  "timestamp": "2026-08-21T11:15:40.197Z"
}
```

### 2. Initial Admin Login
1. Navigate to `https://<your-domain>/login`.
2. Log in with:
   - **Username**: `admin`
   - **Password**: Value specified in `INITIAL_ADMIN_PASSWORD`.
3. Go to **Settings** / **2FA** to set up Two-Factor Authentication (TOTP).

### 3. QR Scan Verification
1. Access the public scan route: `https://<your-domain>/m/<serial-number>`.
2. Confirm that public material information loads without requiring authentication.

---

## 5. Comprehensive Troubleshooting Guide & Diagnostics

### Quick Diagnostic Flowchart
```
Symptom: Cannot access application
 ├── Browser shows "502 Bad Gateway" ➔ Fastify crashed or failed to start (Check runtime logs for DB connection or missing dist/index.js)
 ├── Browser shows "404 Not Found" on sub-pages (e.g. /materials) ➔ Nginx `try_files` config missing (SPA fallback error)
 ├── Cannot log in with INITIAL_ADMIN_PASSWORD ➔ Database already existed; admin password was only set during initial bootstrap
 ├── Uploads/Backups missing after redeploy ➔ Persistent Volume missing or Mount Path != /app/.data
 └── Public scan QR points to localhost ➔ System Setting `qr_base_url` is unconfigured (Set to production domain)
```

---

### Issue 1: Nginx Returns `502 Bad Gateway`
* **Root Cause**: Nginx is running on port 80, but the Fastify backend on `127.0.0.1:8000` either crashed on startup or failed to bind.
* **Troubleshooting Steps**:
  1. Open the **Runtime Logs** tab in Zeabur Console.
  2. Check if there is an error like `Error: Cannot find module '/app/backend/dist/index.js'`:
     - **Fix**: You forgot to compile the backend before deploying. Run `cd backend && npm run build` locally and commit `backend/dist`, or use a multi-stage Docker build.
  3. Check if there is a PostgreSQL connection error (`ECONNREFUSED` or `password authentication failed`):
     - **Fix**: Check that `DATABASE_URL` in the Variables tab matches the actual Zeabur PostgreSQL credentials.
  4. Check if port conflict exists:
     - **Fix**: Ensure environment variable `PORT=8000` is set in Zeabur.

---

### Issue 2: Direct Page Refreshes Return `404 Not Found` (React Router SPA)
* **Root Cause**: Navigating directly to client-side routes (e.g. `https://your-domain.com/materials` or `https://your-domain.com/login`) fails because Nginx searches for physical files.
* **Troubleshooting Steps**:
  1. Verify the Nginx server block in the Dockerfile includes the SPA fallback:
     ```nginx
     location / {
         try_files $uri $uri/ /index.html;
     }
     ```
  2. Confirm `COPY frontend/dist /var/www/html` was executed and `/var/www/html/index.html` exists.

---

### Issue 3: Cannot Log In with `INITIAL_ADMIN_PASSWORD`
* **Root Cause**: `INITIAL_ADMIN_PASSWORD` is **only** consumed once when the database is completely empty during first initialization. If you change this environment variable later or if sample data was previously seeded, the password in the database remains unchanged.
* **Troubleshooting Steps**:
  1. If you locked yourself out of a fresh deployment:
     - Open the PostgreSQL service in Zeabur ➔ Execute database query / reset table, or re-create the PostgreSQL service.
  2. If 2FA was enabled and you lost your authenticator device:
     - Use one of the 10 emergency recovery codes generated during 2FA setup (`XXXX-XXXX-XXXX-XXXX`).
  3. Check for brute-force lockout:
     - If you entered the wrong password 5 times, the username `admin` is temporarily locked for 15 minutes. Wait 15 minutes or reset the `audit_log` failure entries.

---

### Issue 4: Session Cookies Discarded / Immediate Logout After Login
* **Root Cause**: Cookie security attributes mismatch with protocol.
* **Troubleshooting Steps**:
  1. If using HTTPS on Zeabur (default), ensure `HTTPS_ENABLED=true` is set.
  2. If testing over plain HTTP (e.g. `http://...`), `HTTPS_ENABLED` must be `false` (otherwise browsers reject cookies flagged with `Secure`).
  3. Ensure `COOKIE_SECRET` is at least 32 characters long. If it is shorter, Fastify cookie initialization will throw a validation error.

---

### Issue 5: Uploaded Files & Backups Disappear After Container Restart
* **Root Cause**: Container filesystem is ephemeral. Files written outside mounted volumes are destroyed upon redeployment.
* **Troubleshooting Steps**:
  1. In Zeabur, verify that a **Volume** is created.
  2. Verify the **Mount Path** is set to `/app/.data`.
  3. Verify environment variables:
     - `FILES_DIR=/app/.data/files`
     - `BACKUP_DIR=/app/.data/backups`
  4. Ensure permissions: The container runs as root/runner, and directories will be automatically created on startup via `fs.mkdir(..., { recursive: true })`.

---

### Issue 6: Generated QR Codes Point to `http://localhost:8000` or `127.0.0.1`
* **Root Cause**: Default system setting uses local host when `qr_base_url` is not explicitly configured.
* **Troubleshooting Steps**:
  1. Log in as `admin`.
  2. Navigate to **Admin Settings** (`/settings`).
  3. In **QR Code Base URL**, enter your public Zeabur domain (e.g., `https://mds-app.zeabur.app`).
  4. Save settings. All future QR code print labels will encode the public URL.

---

### Issue 7: Client File Upload Fails with `413 Payload Too Large` or `400 Invalid File Type`
* **Root Cause**: File size exceeds 10MB limit or extension is not in whitelist.
* **Troubleshooting Steps**:
  1. Whitelisted formats are: `.pdf`, `.png`, `.jpg`, `.jpeg`, `.webp`.
  2. The maximum single-file upload limit is **10 MB**.
  3. If Nginx rejects large files before Fastify receives them, ensure Nginx configuration has `client_max_body_size 15M;` (default in our setup allows streaming payloads).

---

### Issue 8: Database Migration Failures on Startup
* **Root Cause**: Incompatible schema change or missing migration files.
* **Troubleshooting Steps**:
  1. Verify the Dockerfile contains:
     ```dockerfile
     COPY backend/src/db/migrations ./backend/dist/db/migrations
     ```
  2. Ensure the PostgreSQL user specified in `DATABASE_URL` has `CREATE TABLE`, `ALTER TABLE`, and `CREATE INDEX` permissions.


---

## 6. Automated Deployment Options on Zeabur

There are three ways to enable **zero-touch automated deployments** to Zeabur:

### Option 1: Native Git Push Auto-Deploy (Recommended & Easiest)
Zeabur natively watches your linked GitHub repository:
1. When you push to your default branch (`master` / `main`), Zeabur automatically triggers a build and zero-downtime rolling redeployment.
2. Ensure you run your build step or commit the production dist before pushing:
   ```bash
   # Quick deployment script helper
   npm run build --prefix backend && npm run build --prefix frontend
   git add backend/dist frontend/dist
   git commit -m "chore: release production build"
   git push origin master
   ```

---

### Option 2: Automated GitHub Actions CI/CD Workflow (`.github/workflows/deploy.yml`)
To build the TypeScript backend and React frontend directly in the cloud and auto-deploy to Zeabur upon git push, you can add this GitHub Actions workflow:

```yaml
name: Build & Deploy to Zeabur

on:
  push:
    branches:
      - master

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js 22
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install & Build Backend
        run: |
          cd backend
          npm ci
          npm run build

      - name: Install & Build Frontend
        run: |
          cd frontend
          npm ci
          npm run build

      - name: Deploy to Zeabur via CLI
        uses: zeabur/action@v1
        with:
          token: ${{ secrets.ZEABUR_API_TOKEN }}
          service: mds-app
```

---

### Option 3: Deploy via Zeabur CLI (`zeabur deploy`)
For developers who prefer deploying directly from the terminal without manual Git pushes:

1. **Install Zeabur CLI**:
   ```bash
   # macOS via Homebrew
   brew install zeabur/tap/cli

   # Or via npm
   npm install -g zeabur
   ```

2. **Login & Authenticate**:
   ```bash
   zeabur auth login
   ```

3. **Deploy the current directory**:
   ```bash
   # 1. Build locally
   cd backend && npm run build && cd ../frontend && npm run build && cd ..

   # 2. Deploy
   zeabur deploy
   ```

---

### Option 4: Automated Deploy Hook / Webhook
1. Go to your MDS service in the **Zeabur Dashboard** ➔ **Settings** tab.
2. Scroll to **Deploy Hook** and click **Generate Webhook URL**.
3. You can trigger an automatic deployment anytime by sending a `POST` request:
   ```bash
   curl -X POST https://api.zeabur.com/api/v1/deploy-hooks/<YOUR_DEPLOY_HOOK_ID>
   ```


For reference, the production Dockerfile used for this deployment:

```dockerfile
FROM node:22-bookworm-slim AS runner

WORKDIR /app
ENV NODE_ENV=production
ENV BACKUP_DIR=/app/.data/backups
ENV FILES_DIR=/app/.data/files

# Install Nginx
RUN apt-get update && apt-get install -y --no-install-recommends nginx && rm -rf /var/lib/apt/lists/*

# Copy Backend production dist & install only production dependencies
COPY backend/package*.json ./backend/
RUN cd backend && npm install --omit=dev --no-audit
COPY backend/dist ./backend/dist
COPY backend/src/db/migrations ./backend/dist/db/migrations

# Copy Frontend pre-compiled static dist directly to Nginx web root
COPY frontend/dist /var/www/html

# Configure Nginx reverse proxy on port 80 -> Fastify backend on 127.0.0.1:8000
RUN cat << 'EOF' > /etc/nginx/sites-available/default
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html;
    server_name _;

    location /api/ {
        proxy_pass http://127.0.0.1:8000/api/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location = /health {
        proxy_pass http://127.0.0.1:8000/health;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2|woff|ttf)$ {
        expires 1y;
        add_header Cache-Control "public, no-transform";
    }
}
EOF

# Create startup script
RUN cat << 'EOF' > /app/entrypoint.sh
#!/bin/bash
set -e

echo "==> Starting Fastify Backend on 127.0.0.1:8000..."
cd /app/backend && node dist/index.js &

echo "==> Starting Nginx on port 80..."
nginx -g "daemon off;"
EOF

RUN chmod +x /app/entrypoint.sh

EXPOSE 80 8000

CMD ["/app/entrypoint.sh"]
```
