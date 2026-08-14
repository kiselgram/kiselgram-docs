# Personal Chat & Messages

Direct messaging: send/delete/edit messages, read receipts, typing indicator, emoji reactions,
scheduling and attachments. This is the core of the "personal" `Chat` (`chat_type='personal'`).

## Data models (`app/models.py`)

- **Chat** — `id`, `chat_type` ('personal' | 'group' | 'channel'), `peer1_id`, `peer2_id`
  (the two participants for a personal chat), `created_at`, `last_message_at`.
- **Message** — `id`, `content`, `sender_id`, `receiver_id`, `chat_id`, `timestamp`, `is_read`,
  `read_at`, `delivered_at`, `has_attachment`, `file_type`, `file_name`, `file_path`,
  `file_size`, `thumbnail_path`, `is_encrypted`, `encrypted_content`, `encryption_key_id`,
  `emoji`, `is_deleted`, `deleted_for_all`, `scheduled_at`, `edited_at`, `is_saved`,
  `is_from_telegram`, `telegram_message_id`.
- **EmojiReaction** — `id`, `message_id`, `user_id`, `emoji`, `created_at`.
- **TypingStatus** — `id`, `chat_id`, `user_id`, `is_typing`, `last_update`.

## Endpoints (`app/routes/spav2/chat.py`, `messages.py`)

All prefixed `V2 + '/api'`, i.e. `/api.v2/api/...`:

- `POST /api/api/chat/create` — create or resolve a personal `Chat`.
- `GET  /api/api/chat/history` — paginated message history for a chat (`chat_id`, `page`).
- `POST /api/api/chat/typing` — report typing state.

Messages:
- `POST /api/api/messages/send` — send a text message to `chat_id` / `receiver_id`.
- `DELETE /api/api/messages/{id}` — soft delete (`is_deleted = True`) or `deleted_for_all`.
- `POST /api/api/messages/{id}/edit` — edit text; sets `edited_at`.
- `POST /api/api/messages/{id}/read` — mark read; sets `is_read` / `read_at`.
- `POST /api/api/messages/{id}/reaction` — add an emoji reaction.
- `POST /api/api/messages/{id}/scheduled` — schedule a message via `scheduled_at`.
- `POST /api/api/messages/forward` — forward a message (see Features).

## Frontend modules (`static/js/k/`)
- `chat.js` — render bubbles, send / edit / delete, read receipts, typing bubble.
- `ui.js` — shared web UI helpers used by the chat screens.

## Notes & limits
- Messages support optional end-to-end style encryption fields (`is_encrypted`,
  `encrypted_content`, `encryption_key_id`) but by default send plaintext.
- Attachments are saved to file paths on `file_path`; an uploaded file must exist before a
  message references it (see [Files & Uploads](files-uploads.md)).
- `telegram_message_id` / `is_from_telegram` allow bridging from the Telegram import mode.
- Empty messages are rejected by the API validator.