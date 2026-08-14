# Contacts & Profile

Contact book, blocking, reporting and profile management.

## Data models (`app/models.py`)

- **Contact** — `id`, `owner_id`, `contact_user_id`, `display_name`, `is_blocked`,
  `created_at`.
- **BlockedUser** — `user_id`, `blocked_user_id`, `reason`, `created_at`.
- **Report** — `id`, `reporter_id`, `reported_user_id`, `reported_message_id`, `reason`,
  `status`, `created_at`.
- **User** — profile surface: `username`, `display_name`, `avatar`, `bio`, `status`,
  `phone`, `birthday`, `last_seen`.
- **UserProfile** — extended profile: `id`, `user_id`, `bio`, `avatar`, `phone`, `link`.

## Backend modules (`app/routes/spav2/`)

- `contacts.py` — contacts & search users.
- `block.py` — blocking/unblocking.
- `profile.py` — view & edit own profile.

## Endpoints (`V2 + '/api'` → `/api.v2/api/...`)

Contacts:
- `GET  /api/api/contacts` — list contacts.
- `POST /api/api/contacts/add` — add a contact (`user_id`).
- `DELETE /api/api/contacts/remove` — remove a contact.
- `POST /api/api/contacts/import` — import from phone number list.

Blocking:
- `POST /api/api/block` — block a `user_id`.
- `POST /api/api/unblock` — unblock.
- `GET  /api/api/blocked` — list blocked users.

Reports:
- `POST /api/api/report` — report a user/message with `reason`.

Profile:
- `GET  /api/api/profile/{username}` — public profile.
- `POST /api/api/profile/update` — update `bio` / `avatar` / `display_name` / `phone`.

## Frontend modules (`static/js/k/`)
- `profile.js` — profile view/edit screens.
- `init.js` — holds global constants `V2 = '/api.v2/api'`, `V3 = '/api.v3'` and shared setup.

## Notes
- A blocked user cannot message you; `messages/send` checks `BlockedUser`.
- Reports feed the admin queue (`/api/admin/reports`, see [Admin Panel](admin-panel.md)).
- `user_id` vs `username` — most endpoints accept the numeric `user_id`; profile lookup uses
  the `username`.