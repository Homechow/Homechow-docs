# HomeChow Go-Live Checklist

Run top to bottom in **staging first**, then production
(`oscar-deployment-performance` Recipe 5). Tick each before launch.

## 1. Code & config
- [ ] `django-oscar==4.0.*` pinned exactly; `pip install -r requirements.txt` reproducible
- [ ] `DJANGO_SETTINGS_MODULE=homechow.settings.production`
- [ ] `python manage.py check --deploy` → **0 issues** (security checks pass)
- [ ] `python manage.py makemigrations --check --dry-run` → no changes
- [ ] `lint-imports` → contract kept
- [ ] `pytest` green (conformance + unit + integration + api)
- [ ] OpenAPI snapshot frozen (`docs/openapi-v1.yml`) and snapshot test green
- [ ] Secrets set as env (never in repo): `DJANGO_SECRET_KEY` (≥50 chars),
      `DJANGO_ALLOWED_HOSTS`, DB, `REDIS_URL`, Paystack, Termii, FCM, Maps,
      `SENTRY_DSN`
- [ ] `SITE_DOMAIN` set to the public host (bare domain). Behind Cloudflare it's
      auto-added to `ALLOWED_HOSTS` and trusted for CSRF as `https://…`, so the
      staff console / Oscar dashboard / admin form POSTs work; the proxy trusts
      `X-Forwarded-Proto`/`Host` (set in base). Extra domains via
      `CSRF_TRUSTED_ORIGINS` (comma-separated)

## 2. Data (irreversibles + seed)
- [ ] `AUTH_USER_MODEL=users.User` migrated **before** any data (ADR-001)
- [ ] PostgreSQL 16 + **PostGIS** extension created (ADR-002)
- [ ] `migrate` applied; bootstrap seed present (dish class + attributes,
      source types, ship/pay event types, groups, Nigeria, comms code)
- [ ] Staff users created; Ops/Support/Finance groups assigned

## 3. Infrastructure
- [ ] web (uvicorn/gunicorn) + ws (daphne) + Celery workers
      (queues: comms, payments, dispatch, default) + Celery beat
- [ ] **Beat schedule** running: `expire_unaccepted_orders` (30s),
      `expire_gig_offers` (10s), `sweep_payouts` (15m) + reminders
- [ ] Redis up (cache + channel layer + broker + GEO); cached template loader on
- [ ] S3 buckets: public media + **private KYC bucket** (encrypted, retention
      via `HOMECHOW_KYC_RETENTION` purge job)
- [ ] `ATOMIC_REQUESTS=True`; DB backups + PITR tested (restore drill)

## 4. Integrations (lead-time — start early; integration-register.md)
- [ ] Paystack **live** keys + **Transfers/Payouts approved**; webhook URL
      registered (`/api/v1/payments/webhooks/paystack`), signature secret set
- [ ] Termii sender ID approved; FCM project (per-audience apps)
- [ ] Google Maps key (server-side, IP-restricted) + billing alerts
- [ ] Webhook signature rejection + replay idempotency verified in staging

## 5. Quality gates (the sacred ones)
- [ ] Money-path test green (multi-chef basket → chef-scoped checkout → one
      charge → one order, preview == submit)
- [ ] Drop-spike load test: 1k buyers on a 20-portion drop → **exactly 20
      orders, 0 oversell**, p95 submit < target
- [ ] Payment-exception suite (400 retryable / 502 / redirect)
- [ ] Vendor isolation matrix; PIN-handoff e2e; ledger reconciliation = 0
      discrepancies
- [ ] Fan-out timing within SLA (20k followers < 60s)

## 6. Security / privacy
- [ ] Throttles active (OTP, basket, submit, PIN, location); PIN lockouts
- [ ] Per-endpoint authN/authZ matrix verified; suspended-account gate
- [ ] NDPR: privacy policy, T&Cs (customer/chef/rider), DPAs, **KYC retention
      values** signed off by counsel (open item O3)
- [ ] Sentry receiving events; structured logs shipping; alerts wired
- [ ] If a CSP is set, the staff console (`/staff/`) loads Bootstrap/htmx/Alpine/
      Font Awesome/Hanken Grotesk from CDN, and the zone editor loads Leaflet +
      Leaflet.draw from unpkg and **map tiles from the OpenStreetMap tile
      server** — allow these origins (`script-src`/`style-src`/`img-src`/
      `connect-src`) or self-host the assets

## 7. Edge states (client-coordinated)
- [ ] `GET /config` returns min app versions + maintenance flag
- [ ] Maintenance mode returns 503 (config + webhooks exempt)
- [ ] Force-update / offline / 500 screens verified against `/config`

## 8. Post-launch watch
- [ ] Stuck-order / stuck-job reconciliation alerts
- [ ] Ledger ↔ Paystack settlement reconciliation (daily)
- [ ] Quarterly `oscar-upgrade-strategy` debt review (the fork manifest is the
      audit scope: partner, order, shipping, checkout)
