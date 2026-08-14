---
layout: page
title: Bots
permalink: /components/bots/
---

# Bots

Bot platform: create and run bot "apps" against the live API on behalf of their owners,
plus the webhook surface bots receive updates on.

## Data models (`app/models.py`)

- **BotApp** — `id`, `owner_id`, `name`, `username`, `token`, `description`,
  `allowed_commands`, `is_active`, `created_at`.
- **BotCommand** — `id`, `bot_id`, `command`, `description`, `handler`.
- **BotWebhook** — `id`, `bot_id`, `url`, `secret`, `verified_at`, `is_active`.

## Backend module (`app/routes/bot_webhook.py`)

Registered in `create_app()` with `url_prefix='/api/bots'`.

## Endpoints

- `POST /api/bots/register` — register a new bot app; returns its `token`.
- `POST /api/bots/{id}/command` — define or update a slash command.
- `POST /api/bots/{id}/webhook` — set a webhook `url` + `secret`.
- `POST /api/bots/{id}/webhook/send` — (owner) dispatch data to the bot's webhook.
- `GET  /api/bots/me` — info for the authenticated bot owner's apps.
- Incoming updates hit `POST /api/bots/webhook` (the shared receive endpoint) with the
  bot `token` to authorize.

## Notes
- Authenticated with the bot owner's user token plus `BotApp.token` for webhook calls.
- `allowed_commands` and `BotCommand` entries drive command parsing; unknown commands are
  ignored.
- Webhook delivery is signed with `secret` and only to `is_active` webhooks.