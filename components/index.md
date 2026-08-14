---
layout: page
title: Components
permalink: /components/
---

# Platform Components

Kiselgram is a modular private messenger. Every subsystem of the platform is documented
in its own subpage under `components/`. Each page covers the data models, backend route
modules, definitive HTTP endpoints, frontend JS modules, and hard limits.

All versioned endpoints live under the `/api.v2` blueprint (with a stub `/api.v3`). Auth,
bots, admin, push and files are mounted separately — details are in each page.

## Component map

| Component | Models | Backend | Endpoints page | Frontend JS |
|---|---|---|---|---|
| [Authentication & Sessions](/components/auth-sessions/) | `User`, `UserSession`, `GoogleAccount` | `auth.py`, `login_v3.py`, `qr_login.py`, `oauth.py`, `push.py`, `sessions.py` | `/api/auth`, `/api/auth/oauth`, `/api/auth/qr`, `/api/api/sessions` (Sessions) | `auth.js` |
| [Personal Chat & Messages](/components/chat/) | `Message`, `Chat`, `EmojiReaction`, `TypingStatus` | `chat.py`, `messages.py` | `/api/api/chat`, `/api/api/messages` | `chat.js`, `ui.js` |
| [Groups](/components/groups/) | `Chat` (type `group`), `GroupMember`, `InviteLink`, `GroupTopic` | `groups.py` | `/api/api/groups` | `groups.js` |
| [Channels](/components/channels/) | `Chat` (type `channel`), `ChannelMember`, `ChannelBind` | `channels.py` | `/api/api/channels` | `channels.js` |
| [Stories](/components/stories/) | `Story`, `StoryView`, `StoryLike`, `StoryReaction`, `StoryPrivacy`, `StoryAllowedUser` | `stories.py` | `/api/api/stories` | `stories.js` |
| [Calls & Video (WebRTC)](/components/calls-video/) | `Call`, `VideoCall`, `VideoCallParticipant`, `RTCSettings` | `calls.py` | `/api/api/calls` + `/video/integration/...` | `calls.js`, `webrtc.js` |
| [Premium](/components/premium/) | `UserPremium`, `PremiumPromo`, `PremiumOrder` | `premium.py` | `/premium` and `/api/api/premium` | `premium.js` |
| [Bots](/components/bots/) | `BotApp`, `BotCommand`, `BotWebhook` | `bot_webhook.py` | `/api/bots` | — |
| [Admin Panel](/components/admin-panel/) | `User`, `Report`, `ModLog` | `admin.py` | `/api/admin` | `admin.js` |
| [Features & Extras](/components/features-extras/) | `Poll`, `PollVote`, `Pin`, `SavedMessage`, `UserMusic`, `RecentSearch`, `Referral`, `UserProfile`, `UserSettings` | `features.py`, `pins.py`, `forward.py`, `saved.py`, `music.py`, `search.py`, `referrals.py`, `ksettings.py`, `profile.py` | `/api/api/*` | `features.js`, `profile.js`, `search.js` |
| [Contacts & Profile](/components/contacts-profile/) | `Contact`, `BlockedUser`, `Report` | `contacts.py`, `block.py`, `profile.py` | `/api/api/contacts` | `profile.js`, `init.js` |
| [Files & Uploads](/components/files-uploads/) | `FileUpload`, `Attachment` | `files.py`, `uploads.py` | `/files/...`, `/uploads/...` | `files.js` |
| [JSON Serialization](/components/json-serialization/) | all of the above | serializer helpers in `utils/helpers.py`, `models.py`, `spav2/*` | — | — |

## Main blueprint layout

- `V2 = '/api.v2/api'` and `V3 = '/api.v3'` are global JS constants (see `static/js/k/init.js`).
- The `/api.v2` master blueprint mounts per-feature sub-blueprints, each holding several
  endpoints below (e.g. `/api/api/messages/send`, `/api/api/groups/create`).
- Points outside the versioned blueprint are listed at the bottom of `../api-reference.md`:
  `/api/push`, `/api/admin`, `/video/integration/...`, `/files/...`, `/uploads/...`,
  `/health`, `/stats`, `/endpoints`, `/ping`, `/test/env`, `/test/env/shutdown`.

## Global route helpers

- `@login_required` guards every authenticated endpoint; an expired/invalid token returns
  `401` with a message like *"Session expired. Please log in again."*
- `@admin_required` restricts the /admin surface.
- `@rate_limit(name, ..., max_requests, window)` throttles sensitive actions (login, register,
  OTP, resend).
- All responses are JSON. Lists return `{success: true, items: [...]}` or `{data: [...]}`.