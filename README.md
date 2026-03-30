# Ministry Mapper Analytics

Self-hosted [Umami](https://umami.is) analytics deployment for Ministry Mapper, optimized for shared servers running on [Coolify](https://coolify.io) or any Docker-based platform.

**Defaults are tuned for staging/dev** (~512MB total footprint). See [Resource Presets](#resource-presets) for production values.

## Quick Start

```bash
# 1. Clone the repo
git clone <repo-url> ministry-mapper-analytics
cd ministry-mapper-analytics

# 2. Create your environment file
cp .env.example .env

# 3. Generate secrets and update .env
openssl rand -hex 32  # → APP_SECRET
openssl rand -hex 16  # → POSTGRES_PASSWORD

# 4. Start the stack
docker compose up -d

# 5. Open Umami
open http://localhost:3000
```

**Default login:** `admin` / `umami` — change this immediately after first login.

## Environment Variables

### Required

| Variable | Description |
|----------|-------------|
| `APP_SECRET` | Random secret for session encryption (min 32 chars) |
| `POSTGRES_PASSWORD` | PostgreSQL password |

### Umami Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `UMAMI_VERSION` | `3.0.3` | Umami Docker image tag |
| `UMAMI_PORT` | `3000` | Host port to expose |
| `DISABLE_UPDATES` | `0` | Disable update check (`1` to disable) |
| `REMOVE_TRAILING_SLASH` | `1` | Treat `/page` and `/page/` as same URL |
| `IGNORE_IP` | _(empty)_ | Comma-separated IPs to exclude from tracking |
| `COLLECT_API_ENDPOINT` | _(empty)_ | Custom endpoint to bypass ad blockers (e.g. `/api/collect`) |
| `TRACKER_SCRIPT_NAME` | _(empty)_ | Custom script name to bypass ad blockers (e.g. `custom-stats.js`) |

### PostgreSQL

| Variable | Default | Description |
|----------|---------|-------------|
| `POSTGRES_VERSION` | `16-alpine` | PostgreSQL Docker image tag |
| `POSTGRES_DB` | `umami` | Database name |
| `POSTGRES_USER` | `umami` | Database user |

> **Note:** PostgreSQL is configured with `TZ=UTC` as [recommended by Umami docs](https://umami.is/docs/install) for consistent timestamps.

## Resource Presets

All resource limits are configurable via `.env`. Choose a preset based on your environment, then copy the values into your `.env` file.

Umami is lightweight — a single Node.js process that typically idles at ~200MB RAM and near-zero CPU. PostgreSQL is the heavier component and benefits most from tuning. The presets below account for both services and leave headroom for traffic spikes.

### Dev / Local

For local development, testing, or CI environments with minimal traffic.

```env
# Umami
UMAMI_CPU_LIMIT=0.15
UMAMI_MEMORY_LIMIT=128M
UMAMI_CPU_RESERVATION=0.05
UMAMI_MEMORY_RESERVATION=64M

# PostgreSQL
DB_CPU_LIMIT=0.15
DB_MEMORY_LIMIT=128M
DB_CPU_RESERVATION=0.05
DB_MEMORY_RESERVATION=64M

# PG Tuning (minimal)
POSTGRES_SHM_SIZE=64m
POSTGRES_SHARED_BUFFERS=16MB
POSTGRES_WORK_MEM=512kB
POSTGRES_MAINTENANCE_WORK_MEM=8MB
POSTGRES_EFFECTIVE_CACHE_SIZE=64MB
POSTGRES_MAX_CONNECTIONS=20
```

| | Value |
|---|---|
| **Total RAM cap** | ~256MB |
| **Total CPU cap** | 0.30 vCPU |
| **Best for** | Local Docker, CI pipelines, quick testing |
| **Expected traffic** | Just you |

---

### Staging / Shared Server (default)

These are the defaults in `.env.example` — no changes needed. Ideal when Umami shares a server with other services (e.g. Coolify hosting Ministry Mapper backend alongside analytics).

```env
# Umami
UMAMI_CPU_LIMIT=0.25
UMAMI_MEMORY_LIMIT=256M
UMAMI_CPU_RESERVATION=0.1
UMAMI_MEMORY_RESERVATION=128M

# PostgreSQL
DB_CPU_LIMIT=0.25
DB_MEMORY_LIMIT=256M
DB_CPU_RESERVATION=0.1
DB_MEMORY_RESERVATION=64M

# PG Tuning (conservative)
POSTGRES_SHM_SIZE=128m
POSTGRES_SHARED_BUFFERS=32MB
POSTGRES_WORK_MEM=1MB
POSTGRES_MAINTENANCE_WORK_MEM=16MB
POSTGRES_EFFECTIVE_CACHE_SIZE=128MB
POSTGRES_MAX_CONNECTIONS=30
```

| | Value |
|---|---|
| **Total RAM cap** | ~512MB |
| **Total CPU cap** | 0.50 vCPU |
| **Best for** | Shared VPS/Coolify with other services, staging environments |
| **Expected traffic** | < 10k pageviews/day |

---

### Production / Dedicated Server

For production deployments with moderate traffic on a dedicated or semi-dedicated server.

```env
# Umami
UMAMI_CPU_LIMIT=0.5
UMAMI_MEMORY_LIMIT=512M
UMAMI_CPU_RESERVATION=0.25
UMAMI_MEMORY_RESERVATION=256M

# PostgreSQL
DB_CPU_LIMIT=0.5
DB_MEMORY_LIMIT=512M
DB_CPU_RESERVATION=0.25
DB_MEMORY_RESERVATION=256M

# PG Tuning (balanced)
POSTGRES_SHM_SIZE=128m
POSTGRES_SHARED_BUFFERS=128MB
POSTGRES_WORK_MEM=4MB
POSTGRES_MAINTENANCE_WORK_MEM=64MB
POSTGRES_EFFECTIVE_CACHE_SIZE=256MB
POSTGRES_MAX_CONNECTIONS=50
```

| | Value |
|---|---|
| **Total RAM cap** | ~1GB |
| **Total CPU cap** | 1.0 vCPU |
| **Best for** | Production sites, small-to-medium business |
| **Expected traffic** | 10k–100k pageviews/day |

---

### Production / High Traffic

For high-traffic production sites on a dedicated server with ample resources.

```env
# Umami
UMAMI_CPU_LIMIT=1.0
UMAMI_MEMORY_LIMIT=1G
UMAMI_CPU_RESERVATION=0.5
UMAMI_MEMORY_RESERVATION=512M

# PostgreSQL
DB_CPU_LIMIT=1.0
DB_MEMORY_LIMIT=1G
DB_CPU_RESERVATION=0.5
DB_MEMORY_RESERVATION=512M

# PG Tuning (aggressive)
POSTGRES_SHM_SIZE=256m
POSTGRES_SHARED_BUFFERS=256MB
POSTGRES_WORK_MEM=8MB
POSTGRES_MAINTENANCE_WORK_MEM=128MB
POSTGRES_EFFECTIVE_CACHE_SIZE=512MB
POSTGRES_MAX_CONNECTIONS=100
```

| | Value |
|---|---|
| **Total RAM cap** | ~2GB |
| **Total CPU cap** | 2.0 vCPU |
| **Best for** | Dedicated servers, high-traffic sites |
| **Expected traffic** | 100k+ pageviews/day |

---

### Quick Reference

| Component | Dev | Staging (default) | Production | High Traffic |
|-----------|-----|-------------------|------------|--------------|
| Umami CPU | 0.15 | 0.25 | 0.5 | 1.0 |
| Umami RAM | 128M | 256M | 512M | 1G |
| PG CPU | 0.15 | 0.25 | 0.5 | 1.0 |
| PG RAM | 128M | 256M | 512M | 1G |
| `shared_buffers` | 16MB | 32MB | 128MB | 256MB |
| `max_connections` | 20 | 30 | 50 | 100 |
| **Total RAM** | **~256M** | **~512M** | **~1G** | **~2G** |
| **Traffic** | Testing | < 10k/day | 10k–100k/day | 100k+/day |

## Bypassing Ad Blockers

Some ad blockers block `script.js` or `/api/send`. Umami provides two env vars to work around this:

```env
# Rename the tracker script
TRACKER_SCRIPT_NAME=custom-stats.js

# Use a custom collect endpoint
COLLECT_API_ENDPOINT=/api/collect
```

Then update your tracking script tag accordingly:
```html
<script defer src="https://your-umami.com/custom-stats.js" data-website-id="..." />
```

## Deploying to Coolify

1. Push this repo to GitHub/GitLab.
2. In Coolify, create a new service from **Docker Compose**.
3. Point it to this repository.
4. Add the environment variables from `.env.example` in Coolify's environment settings.
5. Deploy.

> **Note:** Coolify manages ports and routing. You can remove the `ports` section and let Coolify handle it via `SERVICE_URL_UMAMI_3000`.

> **Note:** If Coolify does not respect `deploy.resources`, set CPU/memory limits through Coolify's service resource settings UI instead.

## Monitoring

```bash
docker stats --no-stream $(docker ps --filter "name=umami" -q) $(docker ps --filter "name=db" -q)
```

## PostgreSQL Tuning Reference

| Parameter | What it does | Rule of thumb |
|-----------|-------------|---------------|
| `shared_buffers` | Main data cache | 15–25% of container memory limit |
| `work_mem` | Per-operation sort/hash memory | 1–4MB for small instances |
| `maintenance_work_mem` | VACUUM, CREATE INDEX memory | 16–64MB |
| `effective_cache_size` | Query planner hint (not allocated) | ~50% of container memory limit |
| `max_connections` | Max concurrent DB connections | 30–50 for Umami (low concurrency) |
| `shm_size` | Docker shared memory for PG | Must be ≥ `shared_buffers` |

## Updating Umami

```bash
# Update the version in .env
UMAMI_VERSION=3.1.0

# Pull and restart
docker compose pull umami
docker compose up -d umami
```

## Security Checklist

- [ ] Change default admin password after first login
- [ ] Set strong `APP_SECRET` and `POSTGRES_PASSWORD`
- [ ] Set `IGNORE_IP` to exclude your own IPs
- [ ] Place behind HTTPS reverse proxy (Coolify/Nginx/Caddy)
- [ ] Don't expose PostgreSQL port to the host
- [ ] Regularly update `UMAMI_VERSION` and `POSTGRES_VERSION`

## License

MIT
