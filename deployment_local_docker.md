# Local Docker Deployment Guideline & Operations Manual

This document provides complete instructions for running and maintaining the **Material Data System (MDS)** on local environments using Docker and Docker Compose. It covers both **Full Production Emulation Mode** (Unified Nginx + Fastify Container + PostgreSQL) and **Development Container Mode** (Database in Docker + Vite/Fastify live-reload on host).

---

## 1. Architecture Modes & Port Reference Matrix

### Port Reference Matrix
| Port Number | Service Name | Running Process | Mode | Purpose & Access URL |
| :--- | :--- | :--- | :--- | :--- |
| **`5173`** | **Frontend (Vite Dev Server)** | Node.js (`vite`) | **Mode B (Dev)** | **`http://localhost:5173`** — React Single Page App with Hot-Module-Reload. |
| **`8000`** | **Backend API Server** | Node.js (`Fastify`) | **Mode B (Dev)** / Internal | **`http://localhost:8000`** — REST API endpoints (`/api/v1/*`, `/health`). |
| **`80`** | **Unified Nginx Web Server** | Nginx Container | **Mode A (Prod)** | **`http://localhost`** (or `:80`) — Production entrypoint combining React SPA + API proxy. |
| **`5432`** | **PostgreSQL Database** | Postgres 17 Alpine | **Both Modes** | **`localhost:5432`** — Database storage (`mds_owner` / `mds_app`). |

```
 Mode A: Full Production Emulation (docker-compose.prod.yml)
 ────────────────────────────────────────────────────────────
                [ Browser: http://localhost:80 ]
                               │
                               ▼
        ┌──────────────────────────────────────────────┐
        │  Container: mds-app (Unified Nginx:80)       │
        │  ├── /var/www/html (Static React Dist)       │
        │  └── /api/* -> Fastify Backend:8000          │
        └──────────────────────┬───────────────────────┘
                               │ (Docker Internal Network)
                               ▼
        ┌──────────────────────────────────────────────┐
        │  Container: mds-postgres (Postgres 17:5432)  │
        │  └── Volume: postgres_data                   │
        └──────────────────────────────────────────────┘

 Mode B: Development Live-Reload Mode (startup.sh / startup.bat)
 ────────────────────────────────────────────────────────────────
   Browser ──> http://localhost:5173 (Vite Dev Server on Host)
                     │ (Proxies /api/* to :8000)
                     ▼
               http://localhost:8000 (Fastify Backend on Host)
                     │
                     ▼
               localhost:5432 (PostgreSQL Container in Docker)
```

---

## 2. Prerequisites


Ensure your system meets the following requirements:
1. **Docker Desktop / Docker Engine**: Version 24.0+ installed and running.
2. **Docker Compose**: Version 2.20+ (`docker compose version`).
3. **Node.js & npm** (Only if running Mode B or compiling dist locally): Node.js `v20+` or `v22+`.

---

## 3. Mode A: Full Production Docker Deployment (Recommended)

This mode runs the exact unified multi-tier container as deployed in staging/cloud environments (Nginx serving React SPA on port 80 and reverse-proxying API traffic to Fastify).

### Step 1: Pre-Build Backend and Frontend
Before building the container image, generate the production dist files:
```bash
# 1. Compile backend TypeScript into backend/dist
cd backend && npm install && npm run build && cd ..

# 2. Build frontend React bundle into frontend/dist
cd frontend && npm install && npm run build && cd ..
```

Verify that the following directories exist:
- `backend/dist/`
- `backend/src/db/migrations/`
- `frontend/dist/`

---

### Step 2: Configure Environment (`.env`)
Create or review your `.env` in the root directory:
```env
# Application Secrets
NODE_ENV=production
PORT=8000
DATABASE_URL=postgresql://mds_app:app_password@postgres:5432/mds
COOKIE_SECRET=change_this_to_a_secure_random_32_character_secret_key_12345
HTTPS_ENABLED=false
INITIAL_ADMIN_PASSWORD=Admin1234!
BACKUP_HOOK_TOKEN=my_secure_backup_token_local_2026

# Storage Paths inside container
BACKUP_DIR=/app/.data/backups
FILES_DIR=/app/.data/files
```

---

### Step 3: Launch with Docker Compose
Start the containers in detached mode:
```bash
docker compose -f docker-compose.prod.yml up --build -d
```

Check running status:
```bash
docker compose -f docker-compose.prod.yml ps
```

You should see two healthy services:
- `mds-postgres` (Port `5432:5432`)
- `mds-app` (Port `80:80`)

---

### Step 4: Access Application
- **Frontend & App Entry**: [http://localhost](http://localhost) (or [http://localhost:80](http://localhost:80))
- **Health Check**: [http://localhost/health](http://localhost/health)
- **API Base**: [http://localhost/api/v1](http://localhost/api/v1)
- **Default Admin Login**:
  - Username: `admin`
  - Password: Value set in `INITIAL_ADMIN_PASSWORD` (or `Admin1234!`)

---

## 4. Mode B: Local Development Launcher (Hot-Reload)

If you want live code reloading for development while running PostgreSQL in Docker:

### One-Click Launch (macOS / Linux):
```bash
chmod +x startup.sh
./startup.sh
```

### One-Click Launch (Windows):
Double-click `startup.bat` or run:
```cmd
startup.bat
```

This will automatically:
1. Start `mds-postgres` on port 5432.
2. Run database migrations (`npm run db:migrate`) and seed sample demo data.
3. Start backend (`http://127.0.0.1:8000`) with `tsx watch`.
4. Start frontend (`http://localhost:5173`) with Vite hot-reload.

---

## 5. Database Management & Useful Docker Commands

### 1. View Live Container Logs
```bash
# View all logs in real-time
docker compose -f docker-compose.prod.yml logs -f

# View backend & Nginx application logs only
docker logs -f mds-app

# View Postgres logs
docker logs -f mds-postgres
```

### 2. Run Database Migrations & Seeds Inside Docker
```bash
# Execute schema migration
docker exec -it mds-app sh -c "cd /app/backend && npm run db:migrate"

# Seed sample materials and tracking events
docker exec -it mds-app sh -c "cd /app/backend && npm run db:seed"
```

### 3. Connect Directly to PostgreSQL Database via `psql`
```bash
docker exec -it mds-postgres psql -U mds_owner -d mds
```

Useful SQL inspection commands:
```sql
-- View all tables
\dt

-- Check registered users
SELECT id, username, role, totp_enabled FROM users;

-- View recent audit logs
SELECT id, action, username, occurred_at FROM audit_log ORDER BY occurred_at DESC LIMIT 10;

-- Exit psql
\q
```

### 4. Backup & Restore Data
```bash
# Create manual DB dump
docker exec mds-postgres pg_dump -U mds_owner -d mds > local_backup_$(date +%Y%m%d).sql

# Restore DB dump
cat local_backup_20260819.sql | docker exec -i mds-postgres psql -U mds_owner -d mds
```

### 5. Stop or Reset the Environment
```bash
# Stop containers without losing data
docker compose -f docker-compose.prod.yml down

# Stop containers and WIPE all database data & uploads (Fresh start)
docker compose -f docker-compose.prod.yml down -v
```

---

## 6. Comprehensive Troubleshooting Guide

### Quick Decision Tree
```
Issue: System not responding locally
 ├── "Port 80 is already in use" ➔ Another local web server (Apache, Nginx, or Skype) occupies port 80. (Map host port to 8080)
 ├── "Port 5432 is already in use" ➔ Local PostgreSQL service is running on host machine. (Stop local postgres: `brew services stop postgresql`)
 ├── Nginx 502 Bad Gateway ➔ Fastify backend failed to start (check `docker logs mds-app`)
 ├── Database connection refused ➔ `mds-postgres` container is unhealthy or starting up
 └── File uploads lost on rebuild ➔ Missing docker volume `mds_files_data`
```

---

### Issue 1: Port Conflict on Port 80 (`bind: address already in use`)
* **Cause**: Another service on your host machine (like local Apache, IIS, Nginx, or another Docker container) is using port 80.
* **Fix**: Change the host port mapping in `docker-compose.prod.yml`:
  ```yaml
  ports:
    - "8080:80"  # Now access via http://localhost:8080
  ```

---

### Issue 2: Port Conflict on Port 5432
* **Cause**: You have a standalone PostgreSQL instance running locally on your Mac/Linux/Windows host.
* **Fix**:
  - Stop the local service:
    ```bash
    # macOS
    brew services stop postgresql@16
    # Linux
    sudo systemctl stop postgresql
    ```
  - Or map the Postgres container to another port (e.g. `5433:5432`).

---

### Issue 3: `Nginx 502 Bad Gateway`
* **Cause**: Nginx is running, but the Fastify Node.js process failed or exited.
* **Fix**:
  1. Inspect the application logs:
     ```bash
     docker logs mds-app
     ```
  2. If it says `Cannot find module '/app/backend/dist/index.js'`, ensure you ran `npm run build` in `/backend` prior to running `docker compose up --build`.
  3. Ensure `DATABASE_URL` uses the Docker network host name `postgres` instead of `localhost` or `127.0.0.1` (e.g., `postgresql://mds_app:app_password@postgres:5432/mds`).

---

### Issue 4: Permissions or Missing Data on Mounts
* **Cause**: The Docker volume `mds_files_data` is mounted to `/app/.data/files`, but parent directory `/app/.data` is missing subdirectories.
* **Fix**:
  Ensure permissions allow directory creation. You can inspect the container filesystem with:
  ```bash
  docker exec -it mds-app ls -la /app/.data
  ```

---

### Issue 5: Initial Admin Account Cannot Log In
* **Cause**: Database was already initialized with a previous password or different hash.
* **Fix**:
  To reset the database cleanly:
  ```bash
  docker compose -f docker-compose.prod.yml down -v
  docker compose -f docker-compose.prod.yml up --build -d
  ```
  Then log in with username `admin` and password `Admin1234!` (or your configured `INITIAL_ADMIN_PASSWORD`).
