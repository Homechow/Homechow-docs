# ADR-008: Vendor modelling — chef = Partner + ChefProfile; riders are not partners

- Status: accepted (2026-06-13)
- Key question: How are vendors anchored in Oscar?
  (`oscar-marketplace` Recipe 1; `oscar-model-customisation` Recipe 3 —
  related model keeps the core app undiverged)
- Decision: A chef is an Oscar `partner.Partner` (kept **undiverged**)
  plus a `ChefProfile` related model in `domain/chefs/` carrying identity,
  kitchen geo, state machine, commission rate and denormalized ratings.
  `Partner.users` links the chef's user — Oscar's documented anchor for
  scoped access — even though chefs get **no dashboard login at MVP**
  (ADR-010): `services/scoping.py` keys its querysets on it for the chef
  API today and the dashboard tomorrow. Each drop's `StockRecord` belongs
  to the chef's partner, which is what routes orders, scoping and payouts.
  **Riders are NOT partners** — they fulfil deliveries, not products;
  rider identity lives entirely in `domain/riders/` + `domain/logistics/`.
- Mechanism: related model (data ladder: "related model" rung — no Partner
  divergence, no migration debt on Oscar's table).
- Rejected:
  - Extending `Partner` with chef fields — starts the divergence clock for
    data that is purely ours.
  - A vendor role system parallel to `Partner.users` — documented
    anti-pattern: two sources of truth for the same fact.
  - Modelling riders as partners "for consistency" — riders never own
    stockrecords; forcing them into Partner corrupts strategy/scoping
    semantics.
- Test: `tests/integration/test_vendor_isolation.py` — the marketplace
  isolation matrix (chef A / chef B / staff × products, orders, drops,
  payouts) run through `APIClient`;
  `tests/unit/chefs/test_onboarding.py::test_approval_creates_partner_link`
- Fork-manifest impact: `partner` forked for the strategy only (ADR-005);
  models unchanged
