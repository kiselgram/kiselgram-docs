# Files & Uploads

Media and file upload/download for chat attachments, stories, avatars and profile photos.

## Data models (`app/models.py`)

- **FileUpload** — `id`, `owner_id`, `original_name`, `path`, `size`, `mime_type`,
  `hash`, `created_at`.
- **Attachment** — `id`, `message_id`, `file_upload_id`, `media_type`, `thumbnail_path`,
  `created_at`.
- Messages reference media via `Message.file_path` / `thumbnail_path` (see [Chat](chat.md)).

## Backend modules (`app/routes/`)

- `app/routes/files.py` — `files_bp` at `url_prefix='/files'`.
- `app/routes/uploads.py` — `uploads_bp` at `url_prefix='/uploads'`.

## Endpoints

- `POST /files/upload` (multipart) — upload a file; stores to disk and registers a
  `FileUpload`; returns `{path, size, mime_type}`.
- `GET  /files/{id}` — download a file by id (`@login_required`).
- `GET  /uploads/{path}` — serve an uploaded file static asset by path (used by stories,
  avatars, chat attachments).
- `POST /uploads/multipart` — chunked upload for large files (returns upload `session_id`,
  supports `resume`).
- `POST /uploads/complete` — finalize a chunked upload.

## Frontend module (`static/js/k/`)
- `files.js` — file picker, progress bars, upload-to-chat flow.

## Limits & notes
- File size caps are configured server-side; premium users historically receive a higher cap.
- Uploaded files are stored under the configured upload directory and referenced by relative
  `path` in models; the web server serves `/uploads/{path}`.
- A story/attachment must be uploaded (and its `path` returned) **before** calling
  `stories/create` or `messages/send`.
- MIME type + `hash` are stored to detect duplicates and sanitize content types.