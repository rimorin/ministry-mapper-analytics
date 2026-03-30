# Ministry Mapper Analytics

Self-hosted [Umami](https://umami.is) analytics for Ministry Mapper via Docker. Optimized for shared servers on [Coolify](https://coolify.io) or any Docker platform.

## Quick Start

```bash
git clone <repo-url> ministry-mapper-analytics && cd ministry-mapper-analytics
cp .env.example .env

# Generate secrets and update .env
openssl rand -hex 32  # → APP_SECRET
openssl rand -hex 16  # → POSTGRES_PASSWORD

docker compose up -d
# Access via your Coolify-assigned domain — login: admin / umami (change immediately)
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_SECRET` | — | **Required.** Session encryption secret (min 32 chars) |
| `POSTGRES_PASSWORD` | — | **Required.** PostgreSQL password |
| `UMAMI_VERSION` | `3.0.3` | Umami Docker image tag |
| `UMAMI_PORT` | `3000` | Host port to expose |


| `TRACKER_SCRIPT_NAME` | — | Custom script name to [bypass ad blockers](#bypassing-ad-blockers) |
| `COLLECT_API_ENDPOINT` | — | Custom collect endpoint to [bypass ad blockers](#bypassing-ad-blockers) |
| `POSTGRES_VERSION` | `16-alpine` | PostgreSQL Docker image tag |
| `POSTGRES_DB` | `umami` | Database name |
| `POSTGRES_USER` | `umami` | Database user |

> PostgreSQL uses `TZ=UTC` per [Umami docs](https://umami.is/docs/install) for consistent timestamps.

## Resource Presets

Defaults are tuned for **staging** (~512MB total). Copy values from the preset matching your environment into `.env`. All resource variables (`UMAMI_CPU_LIMIT`, `DB_MEMORY_LIMIT`, `POSTGRES_SHARED_BUFFERS`, etc.) are listed in `.env.example`.

| | Dev | Staging *(default)* | Production | High Traffic |
|---|---|---|---|---|
| **Umami** CPU / RAM | 0.15 / 128M | 0.25 / 256M | 0.5 / 512M | 1.0 / 1G |
| **PostgreSQL** CPU / RAM | 0.15 / 128M | 0.25 / 256M | 0.5 / 512M | 1.0 / 1G |
| `shared_buffers` | 16MB | 32MB | 128MB | 256MB |
| `max_connections` | 20 | 30 | 50 | 100 |
| **Total footprint** | ~256M | ~512M | ~1G | ~2G |
| **Traffic** | Testing only | < 10k/day | 10k–100k/day | 100k+/day |

### PostgreSQL Tuning Guide

| Parameter | Purpose | Rule of Thumb |
|-----------|---------|---------------|
| `shared_buffers` | Main data cache | 15–25% of container RAM |
| `work_mem` | Per-operation sort/hash | 1–4MB for small instances |
| `maintenance_work_mem` | VACUUM, CREATE INDEX | 16–64MB |
| `effective_cache_size` | Planner hint (not allocated) | ~50% of container RAM |
| `max_connections` | Concurrent DB connections | 30–50 for Umami |
| `shm_size` | Docker shared memory | Must be ≥ `shared_buffers` |

## Bypassing Ad Blockers

Some blockers filter `script.js` or `/api/send`. Set custom names in `.env`:

```env
TRACKER_SCRIPT_NAME=custom-stats.js
COLLECT_API_ENDPOINT=/api/collect
```

Then update your script tag:
```html
<script defer src="https://your-umami.com/custom-stats.js" data-website-id="..." />
```

## Deploying to Coolify

1. Push this repo to GitHub/GitLab.
2. In Coolify, create a new service from **Docker Compose** and point it to this repo.
3. Add environment variables from `.env.example` in Coolify's settings.
4. Deploy.

> Coolify manages routing — you can remove the `ports` section and use `SERVICE_URL_UMAMI_3000` instead. If `deploy.resources` is ignored, set limits via Coolify's UI.

## Updating Umami

```bash
# Update UMAMI_VERSION in .env, then:
docker compose pull umami && docker compose up -d umami
```

## Security Checklist

- [ ] Change default admin password after first login
- [ ] Set strong `APP_SECRET` and `POSTGRES_PASSWORD`
- [ ] Place behind HTTPS reverse proxy (Coolify/Nginx/Caddy)
- [ ] Don't expose PostgreSQL port to the host
- [ ] Keep `UMAMI_VERSION` and `POSTGRES_VERSION` up to date

## License

MIT
