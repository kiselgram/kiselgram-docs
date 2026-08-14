# Groups

Group chats: create, invite by link, manage members and roles.

## Data models (`app/models.py`)

- **Chat** — for groups `chat_type='group'`, holding `id`, `title`, `description`, `avatar`,
  `created_by`, `created_at`, `members_count`.
- **GroupMember** — `id`, `chat_id`, `user_id`, `role`
  ('owner' | 'admin' | 'member' | 'participant'), `joined_at`.
- **InviteLink** — `id`, `group_id`, `code` (unique), `link`, `created_by`, `uses`, `created_at`.
- **GroupTopic** (forum mode, optional) — `id`, `chat_id`, `title`, `icon_emoji_id`, `created_at`.

## Endpoints (`app/routes/spav2/groups.py`)

All under `V2 + '/api'` → `/api.v2/api/groups/...`:

- `POST /api/api/groups/create` — create a group (`title`, `description`, optional `avatar`).
- `GET  /api/api/groups/info` — group metadata + member count.
- `GET  /api/api/groups/members` — list members (`chat_id`).
- `POST /api/api/groups/add-member` — add a user by `user_id`.
- `POST /api/api/groups/remove-member` — remove a member.
- `POST /api/api/groups/promote` / `POST /api/api/groups/demote` — set admin role.
- `POST /api/api/groups/transfer` — transfer owner role.
- `POST /api/api/groups/set-title` / `POST /api/api/groups/set-avatar` — update metadata.
- `POST /api/api/groups/leave` — leave the group.
- Invites: `POST /api/api/groups/invite-link/create`, `POST /api/api/groups/invite-link/join`.

## Frontend module (`static/js/k/`)
- `groups.js` — group list, creation form, member management UI.

## Notes
- `InviteLink.code` is unique; `uses` increments each join.
- `GroupMember.role` defaults to `'member'`; the creator is `'owner'`.
- An invite link resolves via `code` before adding the current user as a member.