# ADR-003: API stance — evaluate django-oscar-api, expect custom DRF layer

- Status: accepted (2026-06-13) — **evaluation evidence due in M0**
- Key question: How is Oscar exposed to the three mobile clients?
  (`oscar-drf-api` — approved stance: evaluate `django-oscar-api` first,
  build a custom DRF layer where it doesn't fit; the corpus contains no
  REST guidance, so the four invariants are the contract)
- Decision: Run the mandated evaluation (`oscar-drf-api` Recipe 1) in M0:
  pin a compatible django-oscar-api version, score it against the
  requirements matrix, run the API money-path test. **Expected outcome:
  custom DRF layer in `api/v1/`**, because the gap list is already long:
  phone-OTP JWT auth (the package is session-centric), three audience
  prefixes (customer/chef/rider) with distinct permission models, the
  Paystack payment-action contract, drop-scoped availability semantics,
  chef-scoped checkout extraction (ADR-007), and WebSocket surfaces.
  Either way the four invariants bind: (1) prices/availability only via
  the Strategy; (2) basket identity reproduces the middleware contract;
  (3) placement only through `build_submission` → `SubmissionService`;
  (4) payment failures map the four documented exceptions — no catch-alls.
- Mechanism: evaluation ADR appendix in M0 with the scored gap list; then
  `homechow/api/v1/` per the DRF Integration Blueprint
  (`framework/drf-integration-blueprint.md`). Ladder rung: n/a
  (architecture surface; Oscar seams consumed, never bypassed).
- Rejected:
  - Building custom without running the evaluation — explicit stance
    violation (`oscar-drf-api` anti-pattern list).
  - Adopting django-oscar-api wholesale — its session/auth model and
    checkout shapes would be fought at every audience-specific endpoint.
  - GraphQL — three first-party clients we control; REST + OpenAPI
    snapshot gives contract pinning with far less novelty.
- Test: API money-path parity test (the acceptance test); OpenAPI snapshot
  diff in CI; invariant suite
  (`tests/api/test_invariants.py` — deferred-tax shape, basket hijack,
  tampered totals, exception→HTTP table).
- Fork-manifest impact: none (the API layer rides seams)
