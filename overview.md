---
layout: page
title: Overview
permalink: /overview/
---

# Overview

Kiselgram is a complete, self-hostable messaging platform. It is not a mockup: the production build is a live Flask backend with a JavaScript SPA, deployed on real infrastructure, with a WebRTC video service, a mail server, and an admin panel.

This document gives a high-level overview: what you get, how it is built, and how the pieces fit together.

## Feature set (v4.0.0)

### Messaging
- Real-time 1-on-1 chat with read receipts and typing indicators
- Message editing, deletion (including "delete for all")
- Replies, forwards (with context), archived messages
- Pinned messages (chat-level and per-message)
- Scheduled messages
- GIF search / sending (Tenor)
- Animated emoji (599 animated emojis shipped in `static/stickers/`)
- Reactions, polls, translations

### Groups
- Create / manage public & private groups
- Roles, promotions and demotions (`promote`, `demote`)
- Member management (`remove_member`), leave/join
- Invite link generation, revocation, usage
- Per-group settings (`update_group`)

### Channels
- Create / manage broadcast channels
- Unlimited subscribers (`subscribe`, `unsubscribe`)
- Admin management per channel
- Posting to channels (owner/admins only)

### Stories
- Photo/video stories that expire after 24 hours
- Views, likes, reactions, replies, stats
- Story privacy (allowed users)
- Automatic cleanup of expired stories every 30 minutes

### Calls
- Call history, make/answer/end calls
- WebRTC video rooms (video call service), incoming-call tracking

### Bots & platform
- Bot accounts, per-bot token auth
- Bot webapps
- Incoming webhooks (`/webhook/<token>`)
- Referral system

### Premium
- `UserPremium` model: plan, expiry, auto-renew
- 11 premium fonts, stories, wallpapers, priority support
- Promo-code system (generate / list / toggle / validate)

### Admin
- Full admin panel (`spav2_admin_bp`)
- Users: create, update, delete, set password, toggle admin
- Chats: detail, messages, delete/restore messages, post to chat
- Reports: list, resolve, dismiss, take action
- 2FA / OTP overview and cleanup
- Mail account management (via `mailadmin`)
- Terminal execution (admin-only)
- Promo-code management

### User-facing features (non-admin)
- Contacts (add/remove/rename), blocking/unblocking users
- Global search, in-chat search, recent searches
- Saved messages (with notes), favorites, music library
- Profile editing, avatars (upload + preloaded)
- Appearance: themes, fonts, font size, bubble radius, chat colors, wallpapers
- Notification settings, mute-all, do-not-disturb, per-chat sounds
- Privacy settings (last seen, photo, forwards, calls, messages)
- Sessions management (list & revoke)
- QR login and multi-step (email/OTP/password) login flows
- Google OAuth
- Push notifications via Web Push / VAPID
- Web Push subscription management

## Tech stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.10+, Flask, Flask-SQLAlchemy, Flask-Login, Flask-Migrate, Flask-Mail |
| DB | SQLite (dev) → PostgreSQL 15 (prod via `DATABASE_URL`) |
| Message encryption | AES-256-GCM / Fernet at rest |
| Frontend | Vanilla ES6+ SPA (`static/js/k/`), HTML5/CSS3, fetch() |
| Realtime | Typing/presence endpoints + polling (no external sockets in core app) |
| Video | Separate WebRTC service (`video_server/`), SocketIO |
| Deploy | Gunicorn (prod), Docker Compose, Nginx |
| Email | Docker mailserver + `mailadmin` management service |

## Route systems

Kiselgram ships **two coexisting API generations**:

- **V1** (`app/routes/spa/`) — legacy, session-based, HTML redirects. Prefix `/api/...`.
- **V2** (`app/routes/spav2/`) — JSON API, Bearer-token auth, prefix `/api.v2/api/...`.
- **V3** — a duplicate mount of the same V2 blueprints under `/api.v3/` (compatibility layer for new client builds).

The modern SPA uses V2. Legacy pages/tests still use V1.

## Frontend layout

- Single template shell: `templates/k.html`
- 17+ JS modules in `static/js/k/`, each extending the `window.K` namespace:
  - `chat.js`, `groups.js`, `views.js`, `features.js`, `contacts.js`, `search.js`, `settings.js`, `stories.js`, `music.js`, `calls.js`, `saved.js`, `profile.js`, `modals.js`, `ui.js`, `api.js`, `auth.js`, `init.js`, plus admin/auth/login helpers.
- Pure `fetch()`-based; no framework.
- Also: `k.html`, `free.html`, `mobile.html`, `prem.html` templates for legacy/mobile/premium views.

## Background jobs

- **Story cleanup** — every 30 minutes deletes stories older than 24h and their media (`app/__init__.py`).
- **Bot setup** — on DB init, standard bots are registered (`app/utils/bot_utils.py`).

## Version history

- **v4.0.0** — stories, premium fonts, chat customization, global search, animated emoji
- **v3.0** — groups, channels, file support, bots, video server
- **v2.0** — modern SPA + V2 API

See `NEW.md` in the repository root for release notes.