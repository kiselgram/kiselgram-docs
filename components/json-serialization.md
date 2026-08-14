---
layout: page
title: JSON Serialization
permalink: /components/json-serialization/
---

# JSON Serialization

Exact JSON shapes the V2 API returns for every core entity, taken directly from the
serializer helpers in `app/utils/helpers.py`, `app/models.py` and `app/routes/spav2/*`.
All serializers live server-side; dates are ISO-8601 strings.

## Envelope

Every endpoint returns a wrapper:
- Success: `{"success": true, "data": {...}}` or `{"success": true, "items": [...]}`.
- Error: `{"success": false, "error": {"code": "...", "message": "..."}}`.

## User / Peer

Producer: `_serialize_peer(user)` (`spav2/chat.py:15`) — used in chat list and anywhere a
peer object is embedded; `user_to_dict()` (`utils/helpers.py:208`) and `User.to_dict()`
(`models.py:104`) are the other two variants.

```json
{
  "user_id": 42,
  "username": "alex",
  "display_name": "Alex",
  "avatar_url": "/uploads/avatars/abc.png",
  "is_online": true,
  "last_seen": "2026-08-14T10:00:00",
  "status_emoji": "⭐",
  "is_premium": false,
  "is_bot": false,
  "bot_webapp_url": null
}
```

Helpers variant (`user_to_dict`) additionally exposes: `bio`, `created_at`, `has_story`,
`followers_count`, `following_count`, `groups_count`. `User.to_dict()` (model) exposes:
`bio`, `created_at`, `last_seen`, `is_online`, `status_emoji`, `is_premium` only.

## Message

Two producers:
- `_serialize_message(msg, current_user_id)` (`spav2/messages.py:15`) — full V2 shape.
- `message_to_dict(message, current_user_id)` (`utils/helpers.py:227`) — richer shape (reply
  preview, forwarded_from, reactions map, attachment sizing).

V2 (`_serialize_message`) shape:

```json
{
  "message_id": 1001,
  "sender_id": 42,
  "receiver_id": 7,
  "sender_username": "alex",
  "sender_avatar_url": "/uploads/avatars/abc.png",
  "content": "hello",
  "reply_to_id": null,
  "file_path": null,
  "file_url": "/uploads/abc.txt",
  "file_type": null,
  "is_read": true,
  "timestamp": "2026-08-14T10:00:00",
  "edited_at": null,
  "emoji": null,
  "emoji_file": null,
  "emoji_url": "/static/stickers/animatedemojies/x.webp",
  "emoji_size": 96,
  "emoji_count": 1,
  "reactions": {"🔥": 2, "👍": 1}
}
```

`message_to_dict` differences: uses `id` (not `message_id`), `timestamp_formatted` (HH:MM),
`is_own` (hides sender_id semantics), `reply_to_id`/`reply_to_content` (first 50 chars)/
`reply_to_sender`, `forwarded_from` (original sender name), and for attachments
`formatted_size`, `preview_size` (`big`/`medium`/`none`).

## Chat list item (`spav2/chat.py`)

`GET /api/api/chat_list` (real path `/api.v2/api/chat_list`) mixes three chat shapes.
Common fields: `chat_type`, `last_message`, `unread_count`.

Personal chat:

```json
{
  "chat_type": "personal",
  "peer": {
    "user_id": 7, "username": "maria", "display_name": "Maria",
    "avatar_url": null, "is_online": true, "last_seen": null,
    "status_emoji": "", "is_premium": false, "is_bot": false, "bot_webapp_url": null
  },
  "last_message": {
    "message_id": 1001, "content": "hi", "sender_id": 7,
    "timestamp": "2026-08-14T10:00:00", "is_read": false
  },
  "unread_count": 2
}
```

Saved Messages (self-talk) — `peer` has `username: "saved_messages"` and `display_name:
"Saved Messages"`, plus top-level `is_saved: true`; `unread_count: 0`. Group chat — uses
top-level `group: {group_id, name, avatar_url}` instead of `peer`; `last_message` adds
`sender_username`. Channels follow the same list pattern with `channel` top-level key.

## Group (`_serialize_group`, `spav2/groups.py:13`)

```json
{
  "group_id": 5,
  "name": "Team",
  "description": null,
  "avatar_url": null,
  "owner_id": 1,
  "is_public": true,
  "invite_link": "kiselgram.com/join/HASH",
  "member_count": 3,
  "my_role": "owner",
  "last_message": {
    "message_id": 88, "content": "standup?", "sender_username": "alex",
    "timestamp": "2026-08-14T09:00:00"
  },
  "created_at": "2026-08-01T00:00:00"
}
```

`member_count` is computed via `COUNT(ChatMember)` unless passed in; `my_role` is filled by
the caller from the membership row (`owner`/`admin`/`member`).

## Channel (`spav2/channels.py`)

`GET /api/api/channels/{id}`:

```json
{
  "channel_id": 9,
  "name": "News",
  "description": null,
  "avatar_url": null,
  "owner_id": 1,
  "owner_username": "admin",
  "is_public": true,
  "invite_link": "kiselgram.com/channel/HASH",
  "subscriber_count": 120,
  "is_subscribed": true,
  "admins": [{"user_id": 1, "username": "admin"}],
  "created_at": "2026-08-01T00:00:00"
}
```

Create response is the same without `is_subscribed`/`admins`. Channel messages reuse
`_serialize_message` + a `reactions` count map, with `sender_username`/`sender_avatar_url`.

## Story (`_story_to_dict`, `spav2/stories.py:22`)

```json
{
  "story_id": 33,
  "media_path": "/uploads/stories/abc.mp4",
  "media_type": "video",
  "caption": "holiday",
  "created_at": "2026-08-14T10:00:00",
  "expires_at": "2026-08-15T10:00:00",
  "is_viewed": true,
  "view_count": 12,
  "like_count": 4,
  "my_reaction": "🔥"
}
```

`expires_at = created_at + 24h`. The V1 (`spa/stories.py`) variant returns `views` (list of
viewer user dicts), `likes` (count), `liked` (bool), `my_reaction` (raw reaction string).

## Call (`spav2/calls.py`)

`GET /api/api/calls/history`:

```json
{
  "call_id": 12,
  "call_type": "video",
  "peer": {"user_id": 7, "username": "maria", "avatar_url": null},
  "direction": "outgoing",
  "status": "answered",
  "duration_seconds": 63,
  "created_at": "2026-08-14T10:00:00",
  "ended_at": "2026-08-14T10:01:03"
}
```

`direction` is derived from `caller_id == current_user` ("outgoing" / "incoming").
`POST /api/api/calls/make` returns a fresh call as `{call_id, caller_id, receiver_id,
status: "ringing"}`.

## Session (`spav2/sessions.py`)

`GET /api/api/sessions`: `data.sessions[]`:

```json
{
  "session_token": "abcdef...",
  "device": "iPhone 15",
  "ip_address": "1.2.3.4",
  "is_current": true,
  "last_active": "2026-08-14T10:00:00",
  "created_at": "2026-08-01T00:00:00"
}
```

Response wrapper adds pagination: `page`, `per_page`, `total`, `pages`.
`DELETE /api/api/sessions/{id}` returns `{"success": true, "data": {"message":
"Session terminated"}}`.

## Contact (`spav2/contacts.py`)

`GET /api/api/contacts`:

```json
{
  "contacts": [
    {
      "user_id": 7, "username": "maria", "display_name": "Maria",
      "avatar_url": null, "custom_name": "Masha",
      "is_online": false, "last_seen": null,
      "added_at": "2026-08-01T00:00:00",
      "is_premium": false, "status_emoji": "⭐"
    }
  ],
  "total": 1, "page": 1, "per_page": 50, "pages": 1
}
```

## Search result (`spav2/search.py`)

`GET /api/api/search/global`: `data` is a **single matched user**:

```json
{
  "user_id": 7, "username": "maria", "display_name": "Maria",
  "avatar_url": null, "bio": null,
  "is_online": false, "last_seen": null,
  "is_premium": false, "status_emoji": "",
  "is_bot": false, "bot_webapp_url": null,
  "is_contact": true
}
```

`is_contact` reflects whether the user is in your contact book.

## Settings (`UserKSettings.to_dict`, `models.py:713`)

```json
{
  "user_id": 42,
  "settings": {"theme": "dark", "notifications": true}
}
```

`settings` is a free-form JSON blob stored in `user_k_settings.settings`.

## Reactions / Replies / Forwards (embedded)

- **Reaction** row (`reaction` table): aggregates to `{reaction_type: count}` maps on the
  message — never returned standalone.
- **Reply** (`reply` table): `original_message_id` → `reply_message_id`; materialized into
  `reply_to_id`/`reply_to_content`/`reply_to_sender` on messages.
- **Forward** (`forward` table): `forwarded_message_id` → `forwarded_by_id`,
  `original_sender_name`; materialized into `forwarded_from`.

## File (`models.py:331`, embedded in messages)

Not serialized standalone in V2; embedded as `file_url`/`file_type`/`file_size`/
`formatted_size`/`preview_size`/`thumbnail_path` on a message. Backing row: `File(file_type,
file_name, file_path, file_size, thumbnail_path, preview_size='medium', uploader_id, created_at)`.
`file_url = /uploads/{file_path}`.