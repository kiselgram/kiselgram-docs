# Architecture

This document describes the internal structure of Kiselgram: the application layout, the data model, the two API generations, and how the frontend is organized.

## Repository layout

```
kiselgram-dev/
├── app/
│   ├── __init__.py            # create_app() factory, extensions, security headers, bg jobs
│   ├── config.py              # TOML-backed configuration (config/kis.toml)
│   ├──   models.py              # 42 SQLAlchemy models
│   ├── routes/
│   │   ├── spa/               # V1 legacy session-based API + HTML views
│   │   ├── spav2/             # V2/V3 JSON API (Bearer auth)
│   │   ├── files.py           # Upload/download endpoints
│   │   ├── premium.py         # Premium plan + bot platform endpoints
│   │   ├── premium.json       # Promo-codes store
│   │   ├── utils_api.py       # /health, /stats, /endpoints helpers
│   │   └── video_integration.py  # WebRTC room integration endpoints
│   ├── utils/
│   │   ├── helpers.py         # get_current_user_id, message_to_dict, validate_emoji, ...
│   │   ├── security.py        # security headers, CSRF, rate limiter
│   │   ├── crypto.py          # Fernet (AES-256-GCM) message encryption
│   │   ├── bot_utils.py       # standard bot registration
│   │   └── logging_utils.py   # shared logger wiring
│   ├── uploads/               # uploaded files (dev)
│   └── __main__.py
├── static/
│   ├── js/k/                  # SPA modules (17+ files) extending window.K
│   ├── css/                   # k.css and legacy styles
│   ├── stickers/              # animated emoji + _manifest.json
│   └── ...
├── templates/
│   ├── k.html                 # main SPA shell
│   ├── free.html / mobile.html / prem.html / ...
│   ├── auth/  premium/  profile/  errors/
├── config/
│   └── kis.toml               # runtime configuration
├── video_server/
│   └── app.py                 # WebRTC video service (SocketIO)
├── mailadmin/
│   └── app.py                 # mail account management
├── tests/                     # pytest suite
├── manage.py                  # CLI (start/stop/restart/setup/reset-db/...)
├── wsgi.py                    # Gunicorn entry point
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

## Application factory

`app/__init__.py:create_app()` is the single entry point for the Flask application:

1. Loads `config/kis.toml` through `app.config.Config` (fallback to defaults if missing).
2. Detects production mode: if `DATABASE_URL` is set in the environment (or the host is Ubuntu), the app requires `DATABASE_URL` and `SECRET_KEY`, forces `DEBUG=False`, and enables secure cookies.
3. Initializes extensions: `oauth`, `mail`, `db`, `login_manager`, `migrate`.
4. Registers Google OAuth.
5. Installs a `SecurityHeaders` `after_request` hook and a CSRF context processor.
6. Registers blueprints:
   - `spav2.Api2(app)` → all V2/V3 API modules
   - `spa.register_spa_blueprints(app)` → V1 routes/views
   - `premium_bp`, `files_bp`, `utils_api_bp`
   - conditional: `spav2_admin_bp`, `spav2_push_bp`, `video_int_bp` (if `VIDEO_ENABLED`)
7. `db.create_all()` (safe for concurrent workers).
8. Starts the story-cleanup background thread (runs every 30 minutes).

### Helper endpoints defined in the factory

| Endpoint | Description |
|----------|-------------|
| `GET /health` | `{"status": "ok"}` |
| `GET /api/get_user_id` | Returns current session user id in `X-User-Id` header (204, or 401) |
| `GET /` | Redirects to admin panel if host starts with `admin.`, else serves the landing page |
| `GET /logout` | Clears session, marks user offline |
| `GET /qr/<token>` | QR-login page |
| `GET /join?ref=` | Referral redirect into the SPA |
| `GET /webapp/static` | Bot webapp placeholder |

## Data model

All models are defined in `app/models.py`. Key models and their relationships:

### User & identity
- **User** — username, email (verified flag), display name, password hash, telegram linkage, online/last-seen, admin flag, bot fields, status emoji, privacy settings, appearance settings, Google OAuth fields, soft-delete.
- **UserPremium** — per-user premium: plan, since/expiry, auto-renew, payment method.
- **UserSession** — active login sessions (Bearer tokens).
- **EmailVerification** — email verification tokens.
- **LoginOtp** — one-time passwords for login.
- **QrLoginToken** — QR-login authorization tokens.
- **PushSubscription** — Web Push subscriptions (VAPID).
- **PreloadedAvatar** — server-provided avatars.
- **UserKSettings** — per-user key settings (JSON).

### Chats
- **Chat** — unified chat model. `chat_type` ∈ `personal | group | channel`. Holds name/description/avatar, owner, public flag, invite link, and (for personal) `user1_id`/`user2_id`.
- **ChatMember** — membership & role in a chat (role e.g. `participant`, `admin`, `owner`).
- **ChatSubscriber** — channel subscriptions.
- **GroupPermission** — permission overrides per group/member.
- **ChannelAdmin** — per-channel admin rights.
- **PinnedChat** — user's pinned chats.
- **Archive** — archived chats.
- **InviteLink** — group invite links.

### Messaging
- **Message** — content (encrypted at rest via the `content` property → `crypto.encrypt_message`), sender/chat/receiver, read/edited/scheduled timestamps, attachment fields, animated-emoji fields (`emoji`, `emoji_file`, `emoji_size`, `emoji_count`), soft-delete, forwarded-from.
- **File** — upload metadata (type, name, path, size, preview).
- **Reaction** — per-message reactions (user + reaction_type).
- **Reply** — links `reply_message_id` ↔ `original_message_id`.
- **Forward** — links forwarded message to its original sender.
- **Poll** / **PollVote** — polls and votes.
- **Pin** — message-level pins.

### Stories
- **Story** — media, caption, expiry (24h), music.
- **StoryView**, **StoryLike**, **StoryReaction** — interactions.
- **StoryPrivacy**, **StoryAllowedUser** — story visibility.

### Social / misc
- **Contact**, **BlockedUser**, **Favorite**, **RecentSearch**, **Report**, **Call**, **VideoCall**, **VideoCallParticipant**, **UserMusic**, **SavedMessage**, **Referral**.

> Note: personal chat messages store `chat_id` on a unified `Chat` row; the `receiver_id` column is also populated for convenience.

## Route systems

### V1 (`app/routes/spa/`)

Session/HTML-oriented, mounted under `/` with `/api/...` JSON helpers. Modules:
`auth.py`, `chat.py`, `channels.py`, `groups.py`, `contacts.py`, `messages.py`, `stories.py`, `profile.py`, `calls.py`, `search.py`, `sessions.py`, `pins.py`, `favorites.py`.

### V2 (`app/routes/spav2/`)

JSON API, Bearer-token auth, mounted under `/api.v2/api/...`. Mounted once under `/api.v2` and once under `/api.v3` (stub). Modules:

| Module | Purpose |
|--------|---------|
| `auth.py` | Register, login, logout, verify, check_username |
| `login_v3.py` | Multi-step email/OTP/password login, preloaded avatars |
| `qr_login.py` | QR-login flow (generate/request/status/authorize/login) |
| `oauth.py` | Google OAuth callback/login |
| `chat.py` | Chat list, personal messages, typing, bots |
| `messages.py` | Send/edit/delete, mark read, reactions, typing, emojis |
| `groups.py` | Group CRUD, members, roles, messages, invite links |
| `channels.py` | Channel CRUD, subscribe, admin, messages |
| `features.py` | Polls, pinning, forwards, scheduling, search, translation, gifs, invites, archive/mute/theme, read receipts |
| `contacts.py` | Contacts, block/unblock |
| `stories.py` | Story create/delete/view/like/reaction/reply/stats |
| `profile.py` | Profile, avatar, privacy, settings |
| `search.py` | Global search, in-chat search, recent, user files/music |
| `sessions.py` | List/revoke sessions |
| `saved.py` | Saved messages + notes |
| `music.py` | Music library |
| `ksettings.py` | User settings (k settings) |
| `referrals.py` | Referral info/list/use |
| `calls.py` | Call history/make |
| `push.py` | Web Push subscribe/unsubscribe/VAPID key |
| `bot_webhook.py` | Incoming webhooks for bots |
| `admin.py` | Admin panel (own prefix) |

### Admin (`spav2_admin_bp`)

Served under its own prefix (see [API Reference](api-reference.md) and [Security](security.md)). Admin-only endpoints for users, chats, messages, reports, 2FA, mail, terminal, promos.

## Frontend architecture

The SPA is a single HTML shell (`templates/k.html`) plus JS modules under `static/js/k/`.

- `init.js` defines the global `K` object: `state`, and boots the app. Exposes `window.K`, `window.$`, `window.esc`, `window.V2`, `window.V3`.
- `api.js` — fetch wrapper (`K.api.get/post`).
- Each feature module assigns methods onto `K.<feature>` (e.g. `K.chat`, `K.features`, `K.groups`, `K.music`).
- `features.js` exports a large set of feature functions via `Object.assign(K.features, {...})`.
- Legacy UI (`free.html`, `mobile.html`, `prem.html`) uses `free.js` / `prem.js`.

The frontend talks to `/api.v2/api/...` (V2) endpoints via `fetch()` with Bearer tokens.

## Video call server

`video_server/app.py` runs a SocketIO-based WebRTC signaling service on port 5001. The main app integrates through `video_integration.py` (rooms, create/join/end, embed). Environment: `VIDEO_PORT`, `VIDEO_HOST`, `VIDEO_PRODUCTION`, `VIDEO_EXTERNAL_URL`.

## Background & lifecycle

- **Story cleanup** — `create_app()` spawns a daemon thread that every 30 minutes removes stories older than 24h and deletes their media.
- **Bot registration** — `app/utils/bot_utils.py:setup_bots()` is called during DB init (`manage.py`).
- **Rate limiter** — `app.utils.security.rate_limiter` with per-request cleanup via teardown hook.

## Configuration flow

1. `manage.py` loads `config/kis.toml` for CLI/service settings (ports, logging, video).
2. `app/config.py` maps the TOML into Flask config keys (`SECRET_KEY`, `SQLALCHEMY_DATABASE_URI`, upload limits, feature flags, mail, google, video).
3. Environment variables override in production (`DATABASE_URL`, `SECRET_KEY`, `MESSAGE_ENCRYPTION_KEY`, `MAIL_PASSWORD`).
4. `Config.__init__` applies the TOML-derived dict onto the class attributes.