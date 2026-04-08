# Local Development Setup — Gold Leaf DMS (paperless-ngx fork)

This document covers the local development setup for the `paperless-ngx` fork used in the Gold Leaf Holdings eDMS. It is written for the Serve Digital development team.

---

## Architecture

```
Browser
  ├─► localhost:4200  (ng serve — Angular hot-reload)
  │     Proxies /api, /ws, /accounts, /admin, /static → :8000
  └─► localhost:8000  (goldleaf-dms:dev — Docker image built from this fork)
        ├─ granian ASGI server with --reload (auto-restarts on .py changes)
        ├─ src/ volume-mounted from fork (Python code + Django templates)
        ├─ base.css + logo volume-mounted into collected static
        ├─ PostgreSQL 18
        ├─ Redis 8
        ├─ Gotenberg (Office → PDF)
        └─ Apache Tika (text extraction)
```

The backend runs `goldleaf-dms:dev` — a Docker image built directly from this fork. Our Python code, Django settings, templates, and branding are what actually run. There is no upstream image involved.

---

## Prerequisites

- Docker + Docker Compose v2
- Node.js 18+ and `npx` available

---

## One-Time Setup

### 1. Build the local Docker image

Run from the repo root:

```bash
cd /home/davidokwii/eDMS/paperless-ngx
docker build -t goldleaf-dms:dev .
```

**First build takes ~15–20 minutes** — it installs OCR tools, ML models, Python deps, and runs `collectstatic`. Every subsequent build uses Docker's layer cache and takes seconds for typical code changes.

**Rebuild the image only when:**
- `pyproject.toml` or `uv.lock` changes (Python dependency changes)
- Anything in the `Dockerfile` or `docker/rootfs/` changes

For all other changes (Python code, templates, CSS, Angular), no image rebuild is needed — see the hot-reload section below.

---

## Starting Local Dev

Two terminals are needed: one for the backend containers, one for the Angular dev server.

### Terminal 1 — Backend

```bash
cd /home/davidokwii/eDMS/paperless-ngx/docker/compose
docker compose -f docker-compose.postgres-tika.yml up -d
```

Check it's healthy:

```bash
docker ps --filter name=webserver --format "{{.Names}}: {{.Status}}"
# Should show: ..._paperless-webserver-1: Up X minutes (healthy)
```

### Terminal 2 — Angular dev server

```bash
cd /home/davidokwii/eDMS/paperless-ngx/src-ui
npx ng serve --host 0.0.0.0 --port 4200
```

Wait for `✔ Compiled successfully.` then open **http://localhost:4200**.

---

## What Auto-Reloads

| What you change | How it reloads |
|---|---|
| Angular components, SCSS, HTML templates | `ng serve` hot-reloads the browser instantly — no action needed |
| `theme.scss` (global Angular theme) | `ng serve` picks it up automatically |
| `src/documents/static/base.css` | Hard refresh the browser — WhiteNoise reads from disk on every request |
| `src/documents/static/goldleaf_logo.png` | Hard refresh the browser |
| Django templates (`login.html`, `base.html`, etc.) | Hard refresh the browser — Django reads templates from the mounted `src/` |
| Python code (views, models, serializers, settings) | ~1–2s auto-restart — granian detects the `.py` change and restarts the worker |
| Python dependencies (`pyproject.toml` / `uv.lock`) | Rebuild the image (see One-Time Setup above) |

---

## How It Works Internally

### Volume mounts (docker-compose.postgres-tika.yml)

```yaml
volumes:
  # Full Python source — granian watches for .py changes; templates live here too
  - ../../src:/usr/src/paperless/src

  # Static files into WhiteNoise's collected static dir
  - ../../src/documents/static/base.css:/usr/src/paperless/static/base.css
  - ../../src/documents/static/goldleaf_logo.png:/usr/src/paperless/static/goldleaf_logo.png
```

Two separate locations because:
- `/usr/src/paperless/src` — Django source; Python and templates are read directly from here
- `/usr/src/paperless/static` — WhiteNoise's collected static dir; this is what gets served to browsers

### Key environment variables

| Variable | Value | Purpose |
|---|---|---|
| `GRANIAN_RELOAD` | `true` | granian watches `src/` for `.py` changes and auto-restarts |
| `WHITENOISE_AUTOREFRESH` | `true` | Disables WhiteNoise's in-memory file cache; reads static files from disk on every request |
| `PAPERLESS_CSRF_TRUSTED_ORIGINS` | `http://localhost:4200` | Allows Django to accept login POST requests from the Angular dev server (different origin) |

### Angular proxy (proxy.conf.json)

`ng serve` at `:4200` proxies the following paths to the Django container at `:8000`:

| Path | Purpose |
|---|---|
| `/api` | REST API |
| `/ws` | WebSocket (real-time task status) |
| `/accounts` | Django auth (login / logout) |
| `/admin` | Django admin panel |
| `/static` | Static files (CSS, logo, Bootstrap) |

Without the `/static` proxy, the login page CSS breaks — `ng serve` returns `index.html` for unknown paths, causing a MIME type error in the browser.

### API version

| File | `apiVersion` | Notes |
|---|---|---|
| `src-ui/src/environments/environment.ts` | `'9'` | Intentionally downgraded — do not change |
| `src-ui/src/environments/environment.prod.ts` | `'10'` | Matches server default |

---

## Stopping Local Dev

```bash
# Stop Angular dev server
Ctrl+C in Terminal 2

# Stop backend containers (keeps data volumes)
cd /home/davidokwii/eDMS/paperless-ngx/docker/compose
docker compose -f docker-compose.postgres-tika.yml down
```

To also wipe the database and all document data:

```bash
docker compose -f docker-compose.postgres-tika.yml down -v
```

---

## Troubleshooting

### Container fails to start or exits immediately

```bash
docker logs $(docker ps -lq --filter name=webserver)
```

### Login page has no CSS (unstyled)

The `/static` proxy entry is missing or `ng serve` hasn't restarted since `proxy.conf.json` was last changed. Restart `ng serve`.

### Python changes not auto-reloading

Confirm `GRANIAN_RELOAD=true` is set:

```bash
docker exec $(docker ps -q --filter name=webserver) env | grep GRANIAN_RELOAD
```

Watch the logs for reload events:

```bash
docker logs -f $(docker ps -q --filter name=webserver) 2>&1 | grep -E "reload|Modified|Spawning"
```

### Static file change not visible after hard refresh

Confirm `WHITENOISE_AUTOREFRESH=true`:

```bash
docker exec $(docker ps -q --filter name=webserver) env | grep WHITENOISE
```

### Image not found (`goldleaf-dms:dev`)

The image hasn't been built yet. Run:

```bash
cd /home/davidokwii/eDMS/paperless-ngx
docker build -t goldleaf-dms:dev .
```

### Create a superuser (first-time login)

```bash
docker exec -it $(docker ps -q --filter name=webserver) python manage.py createsuperuser
```
