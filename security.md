---
layout: page
title: Security
permalink: /security/
---

# Security

This document describes how authentication, authorization, session management, message privacy and admin-governed features work in Kiselgram, plus operational security best practices.

## Authentication

Kiselgram supports several login flows, all converging on a **session model**:

1. **Email + password** — `POST /api/login`. Passwords are stored as hashes (never plaintext).
2. **Email OTP** — `POST /api/login/send_otp` sends a one-time code by email (or to a linked Telegram account); verified via `POST /api/login/verify_otp`.
3. **Password-first flow** (`login_password`) with optional preloaded avatars for new accounts.
4. **QR login** — a signed/authenticated device displays a QR code; the remote device authorizes it to exchange long-lived session credentials.
5. **Google OAuth** — account linking and OAuth sign-in. OAuth endpoints must be valid for the configured origin (origin check).

### Session tokens

- Each successful login creates a `UserSession` row with a unique `session_token`.
- V2 API requests authenticate with `Authorization: Bearer <session_token>`.
- Web pages use the Flask-signed session cookie (tamper-evident, HTTP-only).
- `UserSession` includes device/OS metadata, IP, `last_seen_at` and a global logout timestamp (`logged_out_at`).

### Account verification

- New accounts are tied to an email; email verification is enforced on signup (`EmailVerification` tokens).
- Usernames are globally unique and validated at registration (`/api/check_username`).

## Authorization & access control

- **Chat access** — message history of a chat is only returned if the requester belongs to the chat (`ChatMember`) or is a subscriber (channels).
- **Roles** — `ChatMember.role` (`participant`/`admin`/`owner`, and for channels the `ChannelAdmin` right set). Admin actions (group promote/demote, channel admins, delete-for-everyone) check `is_admin()` / ownership before mutating.
- **Blocked users** — `BlockedUser` prevents messages between the two accounts; the blocked user can't create a chat or send messages to the blocker.
- **Message deletion** — only the sender or an admin can delete; `delete_for_everyone` additionally requires the user to still be a member.
- **Reports** — users can report chats/messages; only admins can resolve/dismiss/take_action via the admin panel.

## Message privacy

- **Encryption at rest**: messages are encrypted via Fernet (AES-256-GCM) in `app/utils/crypto.py` with `MESSAGE_ENCRYPTION_KEY`. The database stores ciphertext; content is decrypted in-memory only when served.
- **Soft delete** — deleted messages are flagged and hidden, not hard-removed, allowing recovery by admins.
- **Scheduled messages** — stored encrypted until their send time.
- **Media privacy** — uploaded files are served through auth-gated routes; avatars and public files can be marked public.

> **Important**: encryption at rest protects the database files / backups. It does **not** provide end-to-end encryption between users: the server can decrypt messages. Enable transport security (TLS) so ciphertext never leaks in transit either (see Deployment).

## Web security hardening

`app/utils/security.py` and `create_app()` apply by default:

| Control | Details |
|---------|---------|
| Security headers | `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy`, `X-XSS-Protection`, `Content-Security-Policy` (media/img-src self; frame-ancestors 'none' on API) |
| CSRF | CSRF token generation and validation on state-changing session endpoints |
| Rate limiting | `rate_limiter` on auth endpoints (login, register, OTP, QR) to slow brute force |
| Sessions | HTTP-only cookies; `SameSite=Lax`; `Secure` when production (`secure.cookies` / HTTPS) |
| Content length | Upload size limit (`MAX_CONTENT_LENGTH`, configurable, default 16 MB) |
| Filename sanitization | Uploads are re-encoded/sanitized before writing to disk |

## Admin surface

The admin panel (`spav2_admin_bp`) is the most powerful attack surface. It:

- Has its **own login** and requires an `admin` user.
- Exposes user/chats/message operations, OTP overview & cleanup, mail account management, promo generation, and a **terminal executor**.
- Terminal execution is protected (requires authenticated admin session) but by its nature grants OS-level access — treat the admin host as extremely sensitive: separate host record, MFA at the reverse proxy, no public exposure beyond controlled hosts.

## Privacy controls for users

Each user can configure:

- **Last-seen** (`offline` / last-seen visibility)
- **Profile photo visibility**
- **Forwarding** (disallow forwards of their messages)
- **Calls & contact permissions**
- **Story privacy** — who can see/like stories (`StoryAllowedUser`)

Settings are enforced server-side, not just hidden in the UI.

## Operational security checklist (operators)

1. **Production mode**: set `DATABASE_URL`; app forces `DEBUG=False`, `SECURE_TRANSPORT` enforcement, secure cookies.
2. **Secrets**: `SECRET_KEY` and `MESSAGE_ENCRYPTION_KEY` must be unique, random, and stored in a secret manager / env. Rotate keys with a documented procedure (re-encrypting at rest if key changes).
3. **TLS everywhere**: terminate TLS at Nginx; never serve plaintext in production.
4. **Limit exposed services**: `video_server` and `mailadmin` need not be public; bind them to internal networks.
5. **Backups**: `backup-db` + encrypted volume snapshots. Backups contain ciphertext but also metadata (usernames, contacts, message counts) — protect them at rest with disk encryption.
6. **Monitor**: watch auth endpoints (rate-limit alerts), admin logins, and report volume. Use the `stats`/`endpoints` utilities.
7. **Updates**: pin & regularly update dependencies (`requirements.txt`) — run `pip-audit`.
8. **Least privilege**: for self-host, don't give the app user root; run app, video, mail containers as non-root (see Dockerfile).

## Threat model & known tradeoffs

- **No true E2E encryption** (server-decryptable) — by design; verify whether your threat model requires E2E.
- **Session tokens in DB** — revoking a session requires DB access; lost firewalls on the database are critical.
- **Terminal executor** is OS-level by design — must be firewalled off public networks.
- Security headers are best-effort; CSP for a rich SPA is extensive — review responsiveness on your domain.

See the admin panel docs in [Architecture](/architecture/) and [Deployment](/deployment/) for more.