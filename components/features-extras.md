---
layout: page
title: Features & Extras
permalink: /components/features-extras/
---

# Features & Extras

Polls, pinning, forwarding, saved messages, music, search, referrals, settings and profile —
the "everything else" that enriches the core chat experience. All endpoints below live under
`V2 + '/api'` → `/api.v2/api/...`.

## Data models (`app/models.py`)

- **Poll** — `id`, `question` (≤255), `options` (JSON), `is_multiple`, `is_anonymous`,
  `creator_id`, `chat_type`, `chat_id`, `closed`, `created_at`.
- **PollVote** — `poll_id`, `user_id`, `option_index`, `voted_at`.
- **Pin** — `id`, `user_id`, `chat_type`, `chat_id`, `pinned_at`.
- **SavedMessage** — `id`, `user_id`, `message_id`, `saved_at`.
- **UserMusic** — `id`, `user_id`, `file_url`, `file_name`, `artist`, `title`, `duration`,
  `source_message_id`, `added_at`.
- **RecentSearch** — `id`, `user_id`, `search_query`, `search_type`, `created_at`.
- **Referral** — `id`, `inviter_id`, `invited_user_id` (unique), `created_at`.
- **UserProfile** — `id`, `user_id`, `bio`, `avatar`, `phone`, `link`.
- **UserSettings** — `id`, `user_id`, key/value preferences (theme, notifications, etc.).

## Endpoints, by module

### features.py — polls
- `POST /api/api/poll/create` — body: `question`, `options` (list, 2–10), `is_multiple`,
  `is_anonymous`, `chat_type`, `chat_id`. `question` ≤ 255 chars. Returns poll id.
- `POST /api/api/poll/{id}/vote` — body `option_index`.
- `GET  /api/api/poll/{id}/results` — tally (counts `PollVote` per `option_index`).
- `POST /api/api/poll/{id}/close` — owner closes the poll.

### pins.py
- `POST /api/api/pin` — pin a message/chat (`chat_type`, `chat_id`).
- `DELETE /api/api/pin` — unpin.
- `GET  /api/api/pins` — list your pins.

### forward.py
- `POST /api/api/forward` — forward message `message_id` to `target_chat_id`.

### saved.py
- `POST /api/api/saved/add` — save a message (`message_id`).
- `DELETE /api/api/saved/remove` — unsave.
- `GET  /api/api/saved` — list saved messages.

### music.py
- `POST /api/api/music/add` — add a track (`file_url`, `artist`, `title`, `source_message_id`).
- `DELETE /api/api/music/remove` — remove track.
- `GET  /api/api/music` — your library.

### search.py
- `GET /api/api/search/global` — global search by `q` (min 2 chars), matched with `ilike`
  on `username` / `display_name`; `per_page` capped at 50.
- `GET /api/api/search/history` — recent searches (`RecentSearch`).

### referrals.py
- `GET  /api/api/referrals/me` — your `invite_url`
  (`https://kiselgram.ru/join?ref={username}`) and invited count.
- `POST /api/api/referrals/invite` — invite by `ref` / `user_id`. Rewards after threshold 10.

### ksettings.py — settings
- `GET  /api/api/settings` — user settings.
- `POST /api/api/settings/update` — update preference key/value.

### profile.py — public profile
- `GET  /api/api/profile/{username}` — public profile.
- `POST /api/api/profile/update` — update `bio` / `avatar` / `display_name`.

## Frontend modules (`static/js/k/`)
- `features.js` — poll composer & results, pin/forward actions.
- `profile.js` — own + public profile editing.
- `search.js` — global search UI.

## Hard limits recap
- Poll `options`: **2–10**; `question`: **≤255**.
- Global search `q`: **≥2 chars**; `per_page`: **≤50**.
- Referral reward threshold: **10** invited users.