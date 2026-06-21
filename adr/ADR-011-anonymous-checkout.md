# ADR-011: Anonymous checkout disabled

- Status: accepted (2026-06-13)
- Key question: May guests place orders?
  (Oscar: `OSCAR_ALLOW_ANON_CHECKOUT`; `oscar-checkout` Best Practice 7 —
  anonymous checkout weakens per-user offer caps and needs a guest-order
  lookup strategy)
- Decision: `OSCAR_ALLOW_ANON_CHECKOUT = False`. Browsing (feed, chef
  profiles, drop details, search) is fully anonymous-friendly; basket and
  checkout require an OTP-verified account. Every prototype flow
  authenticates before ordering, delivery requires a verified phone for
  the PIN handoff anyway, and per-user voucher/offer caps plus referral
  attribution only work with identified users.
- Mechanism: setting. Ladder rung: 1.
- Rejected:
  - Guest checkout — no PIN-handoff phone, weakened offer caps
    (documented circumvention), orphaned order history, and the OTP
    signup already costs ~20 seconds.
  - Anonymous server-issued basket tokens at MVP — without guest checkout
    they only serve pre-login carts; deferred until a measured funnel
    need appears (the `oscar-drf-api` Recipe 3 pattern is on the shelf).
- Test: `tests/api/test_auth_matrix.py` — basket/checkout endpoints return
  401 for anonymous callers; browse endpoints return 200;
  settings sanity test pins the flag.
- Fork-manifest impact: none
