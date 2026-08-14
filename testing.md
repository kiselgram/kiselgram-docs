---
layout: page
title: Testing
permalink: /testing/
---

# Testing

The project ships a `pytest` suite under `tests/` covering the V1 and V2 APIs, auth flows, models, security helpers, the bot platform, and the video integration.

## Quick start

```bash
# in the project venv
.venv/bin/python -m pytest tests/ -q
```

Run a single file:

```bash
.venv/bin/python -m pytest tests/test_spav2_messages.py -v
```

Run only V2 tests:

```bash
.venv/bin/python -m pytest tests/ -k spav2
```

## Test harness (`tests/conftest.py`)

- `app` fixture (session scope) — application factory (`create_app()`) with an **in-memory SQLite** DB (`sqlite:///:memory:`), plus `TESTING=True`, `MAIL_SUPPRESS_SEND=True`, `WTF_CSRF_ENABLED=False`.
- `client` fixture (function scope) — `app.test_client()` per test.
- `session` fixture (function scope) — truncates **all tables** (`_clear_all_tables()`) so tests are isolated.
- Ready-made users: `user`, `user2`, `user3`, `admin_user`, `premium_user`; ready-made chat: `personal_chat`.
- Logged-in clients: `logged_in_client`, `logged_in_admin`, `logged_in_premium`, `logged_in_user2` (session-based), plus `login_as(client, user)` helper.
- `clear_rate_limiter` (autouse) — resets the in-memory rate limiter before each test.

## Layout of `tests/`

| File | Covers |
|------|--------|
| `test_auth.py` | register/login/logout, check_username, verify email, sessions |
| `test_api_spa.py` | V1 `/api` JSON helpers (chat list, send, messages) |
| `test_bot_api.py` | Bot platform: list, send, webapps, webhook routing |
| `test_chats.py` | Chat list/history, typing, read receipts |
| `test_client_api.py` | Client-facing V2 API basics, error format |
| `test_files.py` | Upload/download, file metadata, size limits |
| `test_helpers.py`, `test_helpers_ext.py` | `message_to_dict`, `get_current_user_id`, emoji validation, models helpers |
| `test_models.py`, `test_models_ext.py` | Core models + relationships + extended model coverage |
| `test_security.py` | Headers, CSRF, rate limiter, crypto round-trip |
| `test_performance.py` | Sanity benchmarks for hot paths (chat list, message send) |
| `test_spa_groups_channels.py` | V1 groups & channels HTML/JSON views |
| `test_spa_messages_stories.py` | V1 messages + stories |
| `test_spa_profile_calls.py` | V1 profile & calls |
| `test_spav2_auth.py` | V2 auth: check_username, register, login, logout |
| `test_spav2_login_v3.py` | V2 OTP / password / preloaded-avatar login flows |
| `test_spav2_messages.py` | V2 send/edit/delete, emojis, reactions, replies, schedule |
| `test_spav2_chat.py` | V2 chat list, per-user messages, typing |
| `test_spav2_contacts.py` | V2 contacts, block/unblock |
| `test_spav2_groups.py` | V2 groups, roles, invites, join/leave |
| `test_spav2_channels.py` | V2 channels, subscribe, admins |
| `test_spav2_stories.py` | V2 stories create/view/like/react/stats |
| `test_spav2_search.py` | V2 search endpoints |
| `test_spav2_sessions.py` | V2 session list/revoke |
| `test_spav2_profile.py` | V2 profile update/avatar/privacy |
| `test_spav2_saved.py` | V2 saved messages & notes |
| `test_spav2_music.py` | V2 music library |
| `test_spav2_ksettings.py` | V2 key settings |
| `test_spav2_oauth.py` | V2 Google OAuth flow |
| `test_spav2_qr_login.py` | V2 QR-login flow |
| `test_spav2_push.py` | V2 Web Push subscribe/unsubscribe |
| `test_spav2_calls.py` | V2 call history/make |
| `test_spav2_admin.py` | Admin panel endpoints & authorization gates |
| `test_spav2_bot_webhook.py` | Bot webhook ingest |
| `test_video_integration.py` | `/video/integration/*` room lifecycle |
| `test_utils_api.py` | `/health`, `/stats`, `/endpoints`, `get_user_id` |

> File list verified against the repo at v4.0.0 — run `python -m pytest --collect-only tests/` for the authoritative list on newer checkouts.

## Writing tests

Follow existing patterns in `test_spav2_*.py`. The suite authenticates via Flask sessions (`login_as`, `logged_in_client`), not Bearer headers:

```python
from tests.conftest import login_as, personal_chat, user, user2

def test_send_message(client, personal_chat, user):
    login_as(client, user)
    r = client.post("/api.v2/api/send_message",
                    json={"chat_id": personal_chat.id, "text": "hi"})
    assert r.status_code == 200
    assert r.get_json()["success"] is True
```

Guidelines:

1. Use the `client` fixture; never share state between tests (tables are cleared).
2. Register fresh users per test (usernames must be unique).
3. Assert both status codes and `success`/`data`/`error` shapes.
4. For auth-gated routes, assert `401`/`403` when no/invalid token is sent.
5. For security-sensitive behavior (rate limits, CSRF), keep tests independent of shared counters.

## CI integration

No CI config is committed; a typical GitHub Actions workflow:

```yaml
- uses: actions/checkout@v4
- uses: actions/setup-python@v5
  with: { python-version: "3.12" }
- run: pip install -r requirements.txt pytest
- run: python -m pytest tests/ -q
```

## Coverage

Optional coverage report:

```bash
.venv/bin/python -m pytest tests/ -q --cov=app --cov-report=term-missing
```

See [Getting Started](/getting-started/) for env setup and [Deployment](/deployment/) for running the same suite in a container.