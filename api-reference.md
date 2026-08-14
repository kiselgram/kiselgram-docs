# API Reference

Kiselgram exposes a JSON API (V2/V3), a legacy session-based API (V1), plus file upload, premium, bot, and video-integration endpoints.

## Conventions

- **V2** — modern JSON API. Mounted twice: `/api.v2/...` and a stub duplicate under `/api.v3/...` for new clients.
- **V1** — legacy, session-based `/api/...` routes inside `/app` (see below).
- **Auth** — V2 requires a **Bearer token** (`Authorization: Bearer <session_token>` from `UserSession`) or a valid Flask session (the test suite uses sessions).
- **Format** — `{"success": true, "data": ...}` on success; `{"success": false, "error": "..."}` (or `{code, message}`) on failure.

## V2/V3 JSON API — full endpoint list

Blueprints are registered under `/api.v2` (duplicated under `/api.v3`). `url_prefix` of each blueprint is noted; full path = `/api.v2` + blueprint prefix + route.

### Auth (`/api/auth` — `auth.py` + `login_v3.py` + `oauth.py` + `qr_login.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/auth/check_username` | Username availability/format check |
| POST | `/api.v2/api/auth/register` | Create account (email + username) |
| POST | `/api.v2/api/auth/login` | Email/password login |
| POST | `/api.v2/api/auth/logout` | End current session |
| GET/POST | `/api.v2/api/auth/verify` | Verify email |
| POST | `/api.v2/api/auth/check-email` | Check email (login flow) |
| POST | `/api.v2/api/auth/send-otp` | Send OTP (email or Telegram) |
| POST | `/api.v2/api/auth/send-otp-email` | Send OTP by email |
| POST | `/api.v2/api/auth/verify-otp` | Verify OTP |
| POST | `/api.v2/api/auth/login-password` | Password login step |
| POST | `/api.v2/api/auth/login-otp-only` | OTP-only login |
| POST | `/api.v2/api/auth/register-send-code` | Registration step 1: send code |
| POST | `/api.v2/api/auth/register-verify-code` | Registration step 2: verify code |
| POST | `/api.v2/api/auth/register-finish` | Registration step 3: finish (creates user) |
| GET | `/api.v2/api/auth/preloaded-avatars` | Preloaded avatars for signup |
| GET | `/api.v2/api/auth/oauth/<provider>/login` | Start OAuth (provider e.g. `google`) |
| GET | `/api.v2/api/auth/oauth/<provider>/callback` | OAuth callback |
| POST | `/api.v2/api/auth/qr/generate` | Generate QR-login token |
| POST | `/api.v2/api/auth/qr/request` | Request QR-login (device) |
| GET | `/api.v2/api/auth/qr/status/<token>` | Poll QR-login status |
| POST | `/api.v2/api/auth/qr/authorize` | Authorize QR-login |
| POST | `/api.v2/api/auth/qr/login` | Complete QR-login (get session) |

### Chat (`/api` — `chat.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/chat_list` | All chats for the current user |
| GET | `/api.v2/api/messages/<int:user_id>` | Message history with a user |
| GET/POST | `/api.v2/api/bots` | List/create bots |
| GET/PUT | `/api.v2/api/bot/<int:bot_id>/webapp` | Get/update a bot's webapp |
| GET | `/api.v2/api/typing/<chat_type>/<int:chat_id>` | Typing indicator (polling) |

### Messages (`/api` — `messages.py`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api.v2/api/send_message` | Send a personal message (text, file, animated emoji fields) |
| POST | `/api.v2/api/mark_read/<int:user_id>` | Mark read |
| POST | `/api.v2/api/messages/<int:message_id>/edit` | Edit message |
| POST | `/api.v2/api/messages/<int:message_id>/delete` | Delete message (soft) |
| POST | `/api.v2/api/typing/<chat_type>/<int:chat_id>` | Set typing indicator |
| POST | `/api.v2/api/reactions/add` | Add reaction |
| GET | `/api.v2/api/reactions/<int:message_id>` | Reactions of a message |
| GET | `/api.v2/api/emojis` | Animated emojis (from `static/stickers/_manifest.json`) |

### Groups (`/api` — `groups.py` + `features.py` group routes)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/groups` | My groups |
| POST | `/api.v2/api/groups/create` | Create group |
| GET | `/api.v2/api/groups/<int:group_id>` | Group detail |
| GET | `/api.v2/api/groups/<int:group_id>/members` | Member list |
| POST | `/api.v2/api/groups/<int:group_id>/members/<int:user_id>/role` | Set member role |
| POST | `/api.v2/api/groups/<int:group_id>/update` | Update group |
| POST | `/api.v2/api/send_group_message` | Send a group message |
| GET | `/api.v2/api/group_messages/<int:group_id>` | Group history |
| GET | `/api.v2/api/join_group/<invite_link>` | Join via invite link |
| POST | `/api.v2/api/leave_group/<int:group_id>` | Leave group |
| POST | `/api.v2/api/groups/<int:chat_id>/promote` | Promote member |
| POST | `/api.v2/api/groups/<int:chat_id>/demote` | Demote member |
| POST | `/api.v2/api/groups/<int:chat_id>/remove_member` | Remove member |
| GET | `/api.v2/api/groups/<int:chat_id>/invites` | Invite links |
| POST | `/api.v2/api/groups/<int:chat_id>/invites/create` | Create invite |
| POST | `/api.v2/api/groups/<int:chat_id>/invites/revoke` | Revoke invite |

### Channels (`/api` — `channels.py`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api.v2/api/channels/create` | Create channel |
| GET | `/api.v2/api/channels/<int:channel_id>` | Channel detail |
| POST | `/api.v2/api/channels/<int:channel_id>/update` | Update channel |
| POST | `/api.v2/api/channels/<int:channel_id>/subscribe` | Subscribe |
| POST | `/api.v2/api/channels/<int:channel_id>/unsubscribe` | Unsubscribe |
| POST | `/api.v2/api/channels/<int:channel_id>/admins` | Manage admins |
| POST | `/api.v2/api/send_channel_message` | Post to channel |
| GET | `/api.v2/api/channel_messages/<int:channel_id>` | Channel history |

### Contacts (`/api` — `contacts.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/contacts` | Contact list |
| POST | `/api.v2/api/contacts` | Add contact |
| DELETE | `/api.v2/api/contacts/<int:contact_id>` | Remove contact |
| POST | `/api.v2/api/contacts/rename` | Rename contact |
| POST | `/api.v2/api/block_user/<int:user_id>` | Block user |
| POST | `/api.v2/api/unblock_user/<int:user_id>` | Unblock user |
| GET | `/api.v2/api/blocked_users` | Blocked list |

### Stories (`/api` — `stories.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/stories` | Stories list |
| POST | `/api.v2/api/stories/create` | Create story (media + caption) |
| DELETE | `/api.v2/api/stories/<int:story_id>` | Delete own story |
| POST | `/api.v2/api/stories/<int:story_id>/view` | Mark viewed |
| POST | `/api.v2/api/stories/<int:story_id>/like` | Like / unlike |
| POST | `/api.v2/api/stories/<int:story_id>/reaction` | React |
| POST | `/api.v2/api/stories/<int:story_id>/reply` | Reply |
| GET | `/api.v2/api/stories/<int:story_id>/stats` | Views/likes stats |

### Profile (`/api` — `profile.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/profile` | Current profile |
| PUT | `/api.v2/api/profile` | Update profile |
| POST | `/api.v2/api/profile/avatar` | Upload avatar |
| GET | `/api.v2/api/profile/privacy` | Privacy settings |
| GET | `/api.v2/api/profile/settings` | Settings |

### Search (`/api` — `search.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/search/global?q=` | Global search |
| POST | `/api.v2/api/search_in_chat` | Search within a chat |
| GET/POST | `/api.v2/api/recent_searches` | Recent searches |
| GET | `/api.v2/api/users/<int:user_id>/files` | User's public files |
| GET | `/api.v2/api/users/<int:user_id>/music` | User's public music |

### Sessions (`/api` — `sessions.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/sessions` | Active sessions |
| DELETE | `/api.v2/api/sessions/<int:session_id>` | Revoke session |

### Saved / Music / K-settings / Referrals

| Method | Path | Description |
|--------|------|-------------|
| GET/POST | `/api.v2/api/saved_messages` | Saved messages (list/save) |
| POST | `/api.v2/api/saved_messages/<int:saved_id>/note` | Update saved note |
| GET/POST | `/api.v2/api/music/library` | Music library (list/add) |
| DELETE | `/api.v2/api/music/library/<int:track_id>` | Remove track |
| GET/PUT | `/api.v2/api/k/settings` | Key settings (get/update) |
| GET | `/api.v2/api/referrals/info` | Referral stats |
| GET | `/api.v2/api/referrals/list` | Referred users |
| POST | `/api.v2/api/referrals/use` | Apply a referral code |

### Features (`/api` — `features.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/emojis` | Animated emojis list |
| GET | `/api.v2/api/gifs/search` | GIF search |
| POST | `/api.v2/api/messages/forward` | Forward message |
| POST | `/api.v2/api/messages/pin` | Pin message |
| GET | `/api.v2/api/messages/pinned` | Pinned messages |
| POST | `/api.v2/api/messages/pinned/dismiss` | Dismiss pinned |
| POST | `/api.v2/api/messages/schedule` | Schedule a message |
| GET | `/api.v2/api/messages/search` | Search messages |
| GET | `/api.v2/api/messages/search_by_date` | Search by date |
| POST | `/api.v2/api/messages/translate` | Translate a message |
| POST | `/api.v2/api/messages/<int:msg_id>/delete_for_all` | Delete for all |
| GET | `/api.v2/api/messages/<int:msg_id>/read_by` | Read receipts |
| POST | `/api.v2/api/polls/create` | Create poll |
| POST | `/api.v2/api/polls/vote` | Vote in poll |
| POST | `/api.v2/api/chats/<int:chat_id>/archive` | Archive chat |
| POST | `/api.v2/api/chats/<int:chat_id>/mute` | Mute chat |
| POST | `/api.v2/api/chats/<int:chat_id>/unmute` | Unmute chat |
| POST | `/api.v2/api/chats/<int:chat_id>/theme` | Set chat theme |

### Calls (`/api` — `calls.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api.v2/api/calls/history` | Call history |
| POST | `/api.v2/api/calls/make` | Make a call |
| POST | `/api.v2/api/video/create-room` | Create a video room |

---

## Admin API (`/api/admin` — `spav2_admin_bp`)

Registered at the app root (not under `/api.v2`). Requires an admin session.

| Method | Path | Description |
|--------|------|-------------|
| GET/POST | `/api/admin/login` | Admin login |
| GET | `/api/admin/logout` | Admin logout |
| GET | `/api/admin/dashboard` | Dashboard stats |
| GET | `/api/admin/users` | User list |
| POST | `/api/admin/users/create` | Create user |
| POST | `/api/admin/users/<int:user_id>/update` | Update user |
| POST | `/api/admin/users/<int:user_id>/set-password` | Set password |
| POST | `/api/admin/users/<int:user_id>/toggle-admin` | Toggle admin flag |
| POST | `/api/admin/users/<int:user_id>/delete` | Delete user |
| GET | `/api/admin/chats` | Chat list |
| GET | `/api/admin/chats/<int:chat_id>` | Chat detail |
| GET | `/api/admin/chats/<int:chat_id>/messages` | Chat messages |
| POST | `/api/admin/chats/<int:chat_id>/post` | Post into chat |
| POST | `/api/admin/chats/<int:chat_id>/messages/<int:message_id>/delete` | Delete message |
| POST | `/api/admin/chats/<int:chat_id>/messages/<int:message_id>/restore` | Restore message |
| POST | `/api/admin/channels/create` | Create channel |
| GET | `/api/admin/reports` | Reports list |
| POST | `/api/admin/reports/<int:report_id>/resolve` | Resolve report |
| POST | `/api/admin/reports/<int:report_id>/dismiss` | Dismiss report |
| POST | `/api/admin/reports/<int:report_id>/action` | Take action |
| GET | `/api/admin/2fa/overview` | 2FA overview |
| GET | `/api/admin/2fa/otps` | OTP codes list |
| POST | `/api/admin/2fa/cleanup` | Cleanup OTPs |
| GET | `/api/admin/2fa/email-codes` | Email verification codes |
| POST | `/api/admin/2fa/email-codes/cleanup` | Cleanup email codes |
| GET/POST | `/api/admin/mail/accounts` | Mail accounts (list/create) |
| DELETE | `/api/admin/mail/accounts/<path:email>` | Delete mail account |
| POST | `/api/admin/mail/accounts/<path:email>/password` | Set mail password |
| POST | `/api/admin/terminal/exec` | Execute terminal command |
| POST | `/api/admin/promo/generate` | Generate promo code |
| GET | `/api/admin/promo/list` | List promos |
| POST | `/api/admin/promo/toggle/<code>` | Toggle promo |

---

## Push notifications (`/api/push` — `spav2_push_bp`)

Registered at the app root (not under `/api.v2`).

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/push/subscribe` | Save a Web Push subscription |
| POST | `/api/push/unsubscribe` | Remove subscription |
| GET | `/api/push/vapid-public-key` | VAPID public key |

---

## Bot webhooks (`/api/bots` — `bot_webhook.py`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api.v2/api/bots/webhook/<token>` | Incoming webhook for a bot |
| PUT | `/api.v2/api/bots/webhook` | Webhook registration/update |

---

## Files & uploads (`files.py`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/files/upload_file` | Upload a file into a chat (image/document/video) |
| POST | `/files/upload_avatar` | Upload avatar |
| POST | `/files/upload_story` | Upload story media |
| DELETE | `/files/delete_file/<int:message_id>` | Delete a file by its message |
| GET | `/uploads/<path:filename>` | Serve uploaded file |
| GET | `/uploads/avatars/<path:filename>` | Serve avatar |

Allowed extensions are configured via `ALLOWED_IMAGES/ALLOWED_DOCUMENTS/ALLOWED_VIDEOS`; max size via `MAX_CONTENT_LENGTH`.

## Premium & bots (`premium.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | Landing page |
| GET | `/success` | Purchase success page |
| GET | `/api/check/<int:user_id>` | Premium status of a user |
| GET | `/api/features` | Feature flags |
| POST | `/api/validate-promo` | Validate a promo code |
| POST | `/api/activate` | Activate premium (promo) |
| POST | `/api/cancel` | Cancel premium |
| GET | `/api/stickers` | Sticker list |
| GET | `/sticker/<filename>` | Serve a sticker |
| POST | `/api/bot/create` | Create a bot |
| GET | `/api/bot/list` | List bots |
| DELETE | `/api/bot/<int:bot_id>` | Delete a bot |
| POST | `/api/bot/<int:bot_id>/regenerate-token` | Rotate bot token |
| POST | `/api/bot/<token>/send` | Bot sends a message |
| GET | `/api/bot/<token>/updates` | Poll bot updates |
| GET | `/api/bot/<token>/test` | Bot health/test |
| POST | `/admin/generate-promo` | Generate promo (admin) |
| GET | `/admin/promo-codes` | List promos (admin) |
| POST | `/admin/toggle-promo/<code>` | Toggle promo (admin) |

## Video integration (`video_integration.py` — when `VIDEO_ENABLED`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/video/integration/` | Video API root |
| POST | `/video/integration/create-room` | Create a WebRTC room |
| GET | `/video/integration/rooms` | List rooms |
| GET | `/video/integration/room/<room_id>` | Room status |
| GET | `/video/integration/room/<room_id>/info` | Room info |
| GET | `/video/integration/health` | Video service health |
| POST | `/video/integration/leave/<room_id>` | Leave room |
| POST | `/video/integration/call/<int:user_id>` | Call a user |
| GET | `/video/integration/calls/incoming` | Incoming calls |
| POST | `/video/integration/calls/<call_id>/accept` | Accept call |

## Utilities (`utils_api.py`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | `{"status": "ok"}` |
| GET | `/health/detailed` | Detailed health (DB, uptime, routes count) |
| GET | `/stats` | Runtime stats |
| GET | `/ping` | Pong |
| GET | `/endpoints` | Registered V2 endpoints (dev aid) |
| GET/POST | `/test/env` | Env/version info (dev) |
| GET/POST | `/test/env/shutdown` | Shutdown (dev, token-gated) |

## Legacy V1 (`app/routes/spa/`)

Session-based HTML/JSON routes. Auth pages under `/auth/...`; the SPA shell and chat/group/channel pages from the chat blueprint (no prefix); the JSON helpers under `/api/...`.

- **Auth** (`/auth`): `/auth/login`, `/auth/register`, `/auth/check-email`, `/auth/verify/<token>`, `/auth/complete-registration`, `/auth/google`, `/auth/google/callback`, `/auth/logout`, plus JSON `/auth/api/login`, `/auth/api/check_username`, `/auth/api/get_user_id`.
- **Shell/pages**: `/app`, `/a`, `/app_1`, `/k`, `/mobile`, `/premium`, `/kis_info`, `/chat/<chat_id>`, `/group/<group_id>`, `/channel/<channel_id>`, `/@<username>`.
- **Chat JSON** (`/api`): `/api/chat_list`, `/api/messages/<user_id>`, `/api/mark_read/<user_id>`.
- **Groups/Channels/Messages/Calls/Contacts/Stories/Search/Sessions/Favorites/Pins** — further `/api/...` JSON routes in their respective blueprints (`groups.py`, `channels.py`, `messages.py`, `calls.py`, `contacts.py`, `stories.py`, `search.py`, `sessions.py`, `favorites.py`, `pins.py`).