---
layout: page
title: Premium
permalink: /components/premium/
---

# Premium

Subscription tiers, prices, promo codes and premium perk enforcement.

## Data models (`app/models.py`)

- **UserPremium** — `user_id` (PK), `is_premium`, `premium_since`, `premium_expires_at`,
  `premium_auto_renew`, `premium_payment_method`, `premium_plan`.
- **PremiumPromo** — `id`, `code` (unique), `discount_percent`, `max_uses`, `used_count`,
  `expires_at`, `is_active`.
- **PremiumOrder** — `id`, `user_id`, `plan`, `amount`, `currency`, `status`, `created_at`.

## Backend modules

- `app/routes/premium.py` — `premium_bp` with `url_prefix='/premium'`.
- Feature gating lives alongside each feature (e.g. `features.py` checks `user.is_premium`).
- Storage: prices, plans and promo codes are held in `app/routes/premium.toml` /
  `premium.json`. Config path constant: `PREMIUM_CONFIG_PATH = __file__ / 'premium.toml'`.

## Endpoints

Standalone mount `/premium`:
- `GET  /premium/plans` — list plans and RUB prices from the config file.
- `POST /premium/activate` — activate/switch a premium plan (`plan`, optional payment method).
- `POST /premium/redeem` — redeem a promo `code`.
- `GET  /premium/status` — current premium status (`is_premium`, `premium_expires_at`).
- `POST /premium/cancel` — disable auto-renew.

Via versioned API (`V2 + '/api'` → `/api.v2/api/premium/...`):
- `GET  /api/api/premium/status`
- `POST /api/api/premium/activate`

Registered explicitly in `create_app()` alongside the bots blueprint.

## Frontend module (`static/js/k/`)
- `premium.js` — plan picker, prices in RUB, activation and promo-code entry.

## Notes
- Prices are configured in RUB (`premium.toml` / `premium.json`), not hard-coded.
- `UserPremium.is_premium` gates exclusive stickers, larger upload limits, story viewer
  history, etc. (see each feature page).
- Promo codes validate against `PremiumPromo.max_uses` and `expires_at`.