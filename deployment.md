# Deployment

Production deployment is containerized: PostgreSQL, a Gunicorn-backed app, a WebRTC video service, a mail stack, and an Nginx reverse proxy with TLS.

## Production requirements

- Docker + Docker Compose (Compose v2)
- A PostgreSQL 15 (or newer) server (or the bundled `db` service)
- Domain(s) with TLS certificates for `web.`, `api.`, `admin.`, `cdn.`, `docs.` etc. as needed

## Environment

Set in production:

| Variable | Required | Purpose |
|----------|----------|---------|
| `DATABASE_URL` | yes | `postgresql://user:pass@db/kiselgram` (or host DB) |
| `SECRET_KEY` | yes | Flask session/security secret |
| `MESSAGE_ENCRYPTION_KEY` | yes | Fernet key for message-at-rest encryption |
| `VIDEO_PORT`, `VIDEO_HOST`, `VIDEO_PRODUCTION`, `VIDEO_EXTERNAL_URL` | for video | `video_server` wiring |
| `MAIL_PASSWORD` | for mail | SMTP password |

Canonical config lives in `config/kis.toml` (see [Getting Started](getting-started.md#configuration)); env vars override TOML in production.

## Docker Compose topology

`docker-compose.yml` defines the services below. The app auto-detects production mode when `DATABASE_URL` is set.

### services

| Service | Image/base | Exposes | Notes |
|---------|-----------|---------|-------|
| `db` | `postgres:15` | `5432` (internal) | Volume-backed, healthcheck |
| `app` | Dockerfile (python:3.12-slim, Gunicorn) | `8000:8000` (internal) | `wsgi.py`; volume for `uploads/`; depends on `db`; runs migrations on start |
| `video` | `video_server/requirements.txt` | `5001` (internal) | SocketIO WebRTC signaling; needs `VIDEO_EXTERNAL_URL` |
| `mailserver` | Docker mailserver (e.g. `mailserver/docker-mailserver`) | `25, 587, 465` | SMTP/IMAP |
| `mailadmin` | `mailadmin/app.py` | `5060` (internal) | Management UI for mail accounts |
| `nginx` | `nginx:alpine` | `80, 443` | Reverse proxy + TLS termination (mounted certs) |

### Networking & volumes

- All internal services communicate over the compose default network; only `nginx`, `mailserver` and `video` (if public calls needed) are published to the host.
- Persistent data: a named volume for the Postgres data dir and a bind/volume for `app/uploads`.
- The `app` container runs `flask db upgrade` (or `db.create_all` for fresh installs) on startup, so schema is in sync.

## Build & run

```bash
# 1. Build images
docker compose build

# 2. Start everything
docker compose up -d

# 3. First-run: apply schema + seed bots
docker compose exec app python manage.py setup

# 4. Test
curl -sk https://api.<your-domain>/health   # {"status":"ok"}
```

Upgrade:

```bash
git pull
docker compose build app video
docker compose up -d --force-recreate app video
```

## Nginx

Terminate TLS in `nginx/nginx.conf`; proxy:

- `api.<domain>` → `app:8000`
- `web.<domain>` / `www` → `app:8000` (SPA)
- `cdn.<domain>` → `app:8000` for `/uploads/…` (long cache headers)
- `admin.<domain>` → `app:8000` (admin blueprint host routing)
- `call.<domain>` → `video:5001` (WebSocket upgrade for WebRTC)

Make sure `proxy_set_header X-Forwarded-Proto https;` and `Host`/SSL headers are passed so Flask generates correct absolute URLs.

## Running without Docker

Use `manage.py` directly (dev) or Gunicorn (prod):

```bash
# dev
python manage.py start

# production on a single host (system packages) — use gunicorn directly:
gunicorn -w 4 -b 0.0.0.0:8000 --timeout 120 wsgi:app
```

Run `video_server/app.py` alongside when video is needed. Use an external process manager (`systemd` units for app, video, and a worker for cleanup).

## Database & migrations

- Migrations are provided via `flask_migrate` (`migrations/`); on startup the app applies pending ones.
- For greenfield DBs, `db.create_all()` in `manage.py setup` is used; prefer migrations for ongoing changes.
- Backups: Postgres `pg_dump` (or `manage.py backup-db` in single-host dev). Keep at-rest encryption on backups (they include message metadata).

## Logging & monitoring

- `app/utils/logging_utils.py` configures rotating file handlers (app.log, video.log, gunicorn logs).
- `docker compose logs -f app` for live output in the containerized setup.
- `/stats` and `/endpoints` provide runtime introspection (safe-guarded in prod).

## Troubleshooting

| Symptom | Likely cause / fix |
|---------|--------------------|
| App logs `can't connect to db` | `DATABASE_URL` wrong, or `db` still starting — check healthcheck |
| 502 from Nginx | `app` not up; check `docker compose ps` and app logs |
| WebRTC calls fail | `VIDEO_EXTERNAL_URL` must be the reachable public `call.<domain>`, and `nginx` must proxy WebSocket upgrade headers |
| Uploads 413 | `MAX_CONTENT_LENGTH` too low — raise in `config/kis.toml` |
| Secure cookie loop | Terminate TLS properly; ensure `X-Forwarded-Proto` is passed so Flask trusts HTTPS |
| `SECRET_KEY`/`MESSAGE_ENCRYPTION_KEY` unset | App refuses to start in production; set from a secure store |

## Domain layout (reference)

Landing `kiselgram.ru`, web SPA `web.`, API `api.`, admin `admin.`, CDN `cdn.`, desktop downloads `desktop.`, docs `docs.`, help `help.`, calls `call.` — route them in DNS + Nginx accordingly.