---
layout: page
title: Calls & Video (WebRTC)
permalink: /components/calls-video/
---

# Calls & Video (WebRTC)

Voice and video calling plus multi-user WebRTC video rooms.

## Data models (`app/models.py`)

- **Call** — legacy 1:1 call. `id`, `caller_id`, `receiver_id`, `call_type`
  ('audio' | 'video'), `status` ('ringing' | 'answered' | 'ended'), `duration`, `created_at`.
- **VideoCall** — room-based call. `id`, `room_id` (unique), `creator_id`, `call_type`
  (default `'video'`), `status` (default `'active'`), `started_at`, `ended_at`, `duration`,
  `participant_count`.
- **VideoCallParticipant** — `call_id`, `user_id`, `joined_at`, `left_at`, `audio_only`,
  `screensharing`.
- **RTCSettings** — `id`, `user_id`, `stun_server`, `stun_port`, `turn_server`, `turn_port`,
  `turn_username`, `turn_credential`, `enabled`.

## Backend modules

- `app/routes/spav2/calls.py` — call endpoints.
- `app/routes/webrtc.py` — WebRTC config / signaling.
- `app/routes/video/*` — video integration endpoints.

### Video integration external defaults (`app/routes/spav2/calls.py`)
- `VIDEO_DEFAULT_PORT = os.environ.get('VIDEO_PORT', '5001')`
- `VIDEO_TIMEOUT = 5` seconds
- Calls proxy to the video server at the configured host/port.

## Endpoints

Calls (`V2 + '/api'` → `/api.v2/api/calls/...`):
- `POST /api/api/calls/start` — start a voice/video call with a `receiver_id`.
- `POST /api/api/calls/{id}/answer` — answer a ringing call.
- `POST /api/api/calls/{id}/end` — end; sets `status='ended'` and `duration`.

Video rooms:
- `POST /api/api/calls/rooms` — create a `VideoCall` room, returns `room_id`.
- `POST /api/api/calls/rooms/{room_id}/join`
- `POST /api/api/calls/rooms/{room_id}/leave`
- `GET  /api/api/calls/rooms/{room_id}/participants`

External video integration:
- `GET /video/integration/config` — served by the video server for WebRTC config.

## Frontend modules (`static/js/k/`)
- `calls.js` — call UI (ringtone, accept/decline, hang-up).
- `webrtc.js` — media & peer connections using the STUN/TURN from `RTCSettings`.

## Notes
- The video server runs separately; default port `5001`, overridable via `VIDEO_PORT`.
- Only the `Call` endpoints live in the versioned API; room state is mirrored on the video
  server. WebRTC signaling depends on the external host set in the Flask config.