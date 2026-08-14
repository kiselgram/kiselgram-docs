---
layout: page
title: Admin Panel
permalink: /components/admin-panel/
---

# Admin Panel

Platform moderation: manage users, review reports, and broadcast to the whole platform.

## Data models (`app/models.py`)

- **User** — `is_admin` flag and `is_active` control access.
- **Report** — `id`, `reporter_id`, `reported_user_id`, `reported_message_id`, `reason`,
  `status` (default `'pending'`), `created_at`.
- **ModLog** — audit trail of admin actions: `id`, `admin_id`, `action`, `target_id`,
  `details`, `created_at`.

## Backend module (`app/routes/admin.py`)

Mounted outside the versioned API at `url_prefix='/api/admin'`; every endpoint is guarded by
`@admin_required`.

## Endpoints

Users:
- `GET  /api/admin/users` — list/search users with pagination.
- `POST /api/admin/users/{id}/ban` — set `is_active = False` (soft ban).
- `POST /api/admin/users/{id}/unban` — restore access.
- `POST /api/admin/users/{id}/role` — grant/revoke `is_admin`.

Reports:
- `GET  /api/admin/reports` — list reports, filter by `status`.
- `POST /api/admin/reports/{id}/resolve` — mark resolved / dismiss.
- `POST /api/admin/reports/{id}/action` — act on the reported user/message.

Platform:
- `POST /api/admin/broadcast` — send a platform-wide message.
- `POST /api/admin/stats` refresh → see `GET /stats` (public) in extras (global endpoints).

## Frontend module (`static/js/k/`)
- `admin.js` — moderation dashboard: user search, ban/unban, report queue, broadcast box.

## Notes
- Reports always carry a `reason`; `status` values: `pending`, `resolved`, `dismissed`.
- Every action writes a `ModLog` row for auditability.
- The `/api/admin` surface is fully separate from `/api.v2`; there is no `/api.v2/api/admin`.