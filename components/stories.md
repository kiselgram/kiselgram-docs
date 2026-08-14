---
layout: page
title: Stories
permalink: /components/stories/
---

# Stories

Ephemeral media posts (photos/videos) showing a ring on your own profile for 24 hours,
with viewers, likes, reactions and privacy targeting.

## Data models (`app/models.py`)

- **Story** — `id`, `user_id`, `media_path`, `media_type` (default `'image'`), `caption`,
  `music_path`, `privacy_type` (default `'everyone'`), `created_at`.
- **StoryView** — `story_id`, `viewer_id`, `viewed_at` (who has seen it).
- **StoryLike** — `story_id`, `user_id`, `created_at`.
- **StoryReaction** — `story_id`, `user_id`, `reaction`, `created_at`.
- **StoryPrivacy** — `story_id`, `privacy_type` (override).
- **StoryAllowedUser** — `story_id`, `user_id` (explicit allow-list).

## Endpoints (`app/routes/spav2/stories.py`)

All under `V2 + '/api'` → `/api.v2/api/stories/...`:

- `POST /api/api/stories/create` — create a story (`media_path`, `caption`, `privacy_type`).
- `GET  /api/api/stories/feed` — your stories + friends' active stories.
- `GET  /api/api/stories/{id}` — fetch one story.
- `DELETE /api/api/stories/{id}` — delete your story.
- `POST /api/api/stories/{id}/view` — register a view (creates `StoryView`).
- `POST /api/api/stories/{id}/like` / `/unlike` — toggle like.
- `POST /api/api/stories/{id}/reaction` — add an emoji reaction.
- `POST /api/api/stories/{id}/views` — list viewers of your story.

## Frontend module (`static/js/k/`)
- `stories.js` — story ring on profile, full-screen viewer, like & reaction bar.

## Notes & limits
- Story media must already exist in `/uploads/...` before `create`.
- Privacy is resolved as: explicit `StoryAllowedUser` > `privacy_type` ('everyone' handled
  directly, others via `StoryPrivacy`).
- A viewer who is not allowed is blocked from `view`.