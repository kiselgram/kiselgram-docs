# Authentication & Sessions

How users register, log in, stay authenticated, get push tokens and manage active sessions.

## Data models (`app/models.py`)

- **User** — `id`, `username`, `email`, `password_hash`, `display_name`, `avatar`, `bio`,
  `status`, `is_active`, `is_admin`, `is_premium`, `last_seen`, `phone`, `two_factor_secret`,
  `birthday`, `created_at`, `last_activity`.
- **UserSession** — `id`, `user_id`, `session_token` (unique), `device`, `ip_address`,
  `user_agent`, `created_at`, `last_activity`, `is_active`.
- **GoogleAccount** — link table for Google OAuth identity (`google_id`, `user_id`, `email`).

## Backend modules (`app/routes/spav2/`)

- `auth.py` — core registration & password login.
- `login_v3.py` — the newer login flow (`login_v3`).
- `qr_login.py` — QR-code based session login.
- `oauth.py` — Google OAuth exchange.
- `push.py` — push notification tokens.
- `sessions.py` — session list / revoke.

## Endpoints

### Register & login
- `POST /api/auth/register` — create a user. Rate limited with
  `@rate_limit('register', max_requests=5, window=300)`. Username rules enforced:
  *"Username must be 3-32 characters (letters, numbers, underscores)"*.
- `POST /api/auth/login` — password login, returns a `session_token`.
- `POST /api/auth/login_v3` — new login flow (JSON).
- `POST /api/auth/qr` — QR login endpoint.
- `POST /api/auth/logout` — invalidates the current token.
- `GET  /api/auth/me` — current user profile (`@login_required`).

### Google OAuth
- `POST /api/auth/oauth/google` — accept an OAuth code / id_token and create/link an account.

### Push
- `POST /api/push/register` — register `expo_push_token` / FCM token for the current user.
- `POST /api/push/send` — (admin/internal) send a push notification.

## Frontend modules (`static/js/k/`)
- `auth.js` — register / login / logout forms, handles the returned token and stores it.

## Limits & notes
- Passwords are stored only as `password_hash` (bcrypt), never plaintext.
- `UserSession.session_token` is unique; suspending an account or revoking a session flips
  `is_active = False`, which fails `@login_required` with a `401`.
- Login/register are throttled by the global rate limiter (see global helpers in `index.md`).