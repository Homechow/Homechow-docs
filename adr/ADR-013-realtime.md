# ADR-013: Realtime — Django Channels with REST polling fallback

- Status: accepted (2026-06-13)
- Key question: How do live surfaces (tracking, gig offers, stock
  countdowns) reach the apps? (No Oscar coverage — pure project
  infrastructure; chat is deferred per the review, so chat is NOT a
  driver)
- Decision: Django Channels 4 + `channels-redis`, served by Daphne/uvicorn
  alongside the WSGI web processes. Channels: `/ws/orders/{n}` (customer
  tracking: status timeline, rider GPS, ETA, delivery-PIN reveal),
  `/ws/chef` (new-order alerts with sound, order events, rider-arriving),
  `/ws/rider` (gig offers with 15s TTL, job updates, earnings toasts),
  `/ws/drops/{id}` (stock countdown, sold-out flip). Services never touch
  Channels directly — they call `realtime/events.publish(...)`, so web,
  workers and consumers stay symmetrical. **Every WS feature has a REST
  polling fallback** (mobile networks in Lagos drop sockets); gig offers
  additionally ride FCM push so an offline-socket rider still gets the
  alert. WS auth: JWT (header where the client supports it, else
  short-lived token query param).
- Mechanism: ASGI `ProtocolTypeRouter`; Redis channel layer; consumer
  contract tests. Ladder rung: n/a (infrastructure beside Oscar).
- Rejected:
  - Polling-only — the 15s gig-offer ring and FOMO stock countdowns are
    product-defining; polling at the needed frequency costs more than
    sockets.
  - Firebase Realtime Database / third-party realtime — splits the source
    of truth for order state out of the backend that owns the pipeline.
  - Server-sent events — no bidirectional path for location ingest;
    Channels covers both.
- Test: consumer contract tests per channel (auth required, event
  envelope, payload pins); dispatch simulation asserts offer delivery via
  WS *and* FCM stub; tracking polling-fallback parity test (same payload
  shape via REST).
- Fork-manifest impact: none
