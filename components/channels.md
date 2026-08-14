---
layout: page
title: Channels
permalink: /components/channels/
---

# Channels

One-way broadcast channels: create, subscribe, publish posts.

## Data models (`app/models.py`)

- **Chat** — for channels `chat_type='channel'`, with `id`, `title`, `description`, `avatar`,
  `created_by`, `is_verified`, `created_at`, `subscribers_count`.
- **ChannelMember** — `id`, `channel_id`, `user_id`, `role`
  ('owner' | 'admin' | 'subscriber'), `joined_at`.
- **ChannelBind** — `id`, `channel_id`, `group_id`, `created_at` (links a discussion group).
- Posts reuse the **Message** model with `chat_id` pointing at the channel.

## Endpoints (`app/routes/spav2/channels.py`)

All under `V2 + '/api'` → `/api.v2/api/channels/...`:

- `POST /api/api/channels/create` — create a channel (`title`, `description`, `avatar`).
- `GET  /api/api/channels/info` — channel metadata + subscriber count.
- `GET  /api/api/channels/subscribers` — list subscribers.
- `POST /api/api/channels/subscribe` / `POST /api/api/channels/unsubscribe` — toggle membership.
- `POST /api/api/channels/set-title` / `POST /api/api/channels/set-avatar` — update metadata.
- `POST /api/api/channels/verify` — mark channel verified (admin only).
- Publishing posts goes through `messages/send` with the channel `chat_id`.

## Frontend module (`static/js/k/`)
- `channels.js` — channel browsing, subscribe buttons, post composition.

## Notes
- `ChannelMember.role` distinguishes `owner`/`admin` from plain `subscriber`.
- Only `owner`/`admin` can publish; `messages/send` checks the sender's role in the channel.
- Verification is restricted to `@admin_required`.