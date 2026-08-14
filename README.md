# Kiselgram Documentation

Welcome to the Kiselgram documentation. Kiselgram is a self-hosted, modern messaging platform with personal chats, groups, broadcast channels, stories, video calls, bots, and premium features — built with Flask (Python 3.10+), SQLAlchemy, and a vanilla-JavaScript SPA.

## Contents

| Document | What it covers |
|----------|----------------|
| [Overview](overview.md) | What Kiselgram is, feature list, tech stack, domain layout |
| [Getting Started](getting-started.md) | Install, configure, and run locally; `manage.py` reference |
| [Architecture](architecture.md) | Application structure, data model, route systems, frontend layout |
| [API Reference](api-reference.md) | Full endpoint list for the V1 and V2/V3 JSON APIs |
| [Security](security.md) | Authentication, message encryption, sessions, privacy, admin |
| [Deployment](deployment.md) | Production Docker deployment, services, environment variables |
| [Testing](testing.md) | Test suite layout and how to run it |
| [Components](components/index.md) | Per-component deep dives: chat, groups, channels, stories, calls, premium, bots, admin, extras, files |

## Component docs

Each platform subsystem has its own page under [`components/`](components/):

- [Component map](components/index.md) — how everything fits together, blueprint layout, helpers
- [Authentication & Sessions](components/auth-sessions.md) — register/login, OAuth, QR login, push, session management
- [Personal Chat & Messages](components/chat.md) — direct messaging, read receipts, reactions, encryption fields
- [Groups](components/groups.md) — membership, roles, invite links, forum topics
- [Channels](components/channels.md) — broadcast channels, subscriptions, verification
- [Stories](components/stories.md) — 24h media posts, views, likes, reactions, privacy
- [Calls & Video (WebRTC)](components/calls-video.md) — 1:1 calls and multi-user video rooms
- [Premium](components/premium.md) — plans, RUB pricing, promo codes
- [Bots](components/bots.md) — bot apps, commands, webhooks
- [Admin Panel](components/admin-panel.md) — moderation, reports, broadcast
- [Features & Extras](components/features-extras.md) — polls, pins, forwards, saved, music, search, referrals, settings
- [Contacts & Profile](components/contacts-profile.md) — contact book, blocking, reports, profile
- [Files & Uploads](components/files-uploads.md) — uploads, downloads, chunked upload, attachments

## Quick facts

- **Version** — 4.0.0
- **Backend** — Flask, SQLAlchemy
- **Database** — SQLite (development), PostgreSQL 15 (production)
- **Frontend** — Vanilla JavaScript (ES6+) SPA, no framework lock-in
- **Auth** — Bearer tokens (`UserSession.session_token`) with Flask-session fallback
- **Message encryption** — AES-256-GCM (Fernet) at rest via `MESSAGE_ENCRYPTION_KEY`
- **Video calls** — WebRTC, separate `video` service
- **License** — Proprietary (see `LICENSE` in the repository)

## Domain layout

```
kiselgram.ru              ─ Landing page
web.kiselgram.ru          ─ SPA web app
api.kiselgram.ru          ─ API backend
admin.kiselgram.ru        ─ Admin panel
cdn.kiselgram.ru          ─ Uploaded files
desktop.kiselgram.ru      ─ Desktop downloads
docs.kiselgram.ru         ─ Documentation
help.kiselgram.ru         ─ Help center
call.kiselgram.ru         ─ Video call rooms
```

## Sources of truth

The documentation describes the codebase at `kiselgram-dev` v4.0.0. Validate any claim against the source in `app/`, `static/js/k/`, and `tests/`.