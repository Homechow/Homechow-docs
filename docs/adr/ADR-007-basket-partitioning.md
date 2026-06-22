# ADR-007: Multi-chef basket, per-chef checkout

- Status: accepted (2026-06-13) — clarified in review 2026-06-12: orders
  can never span chefs
- Key question: Can a basket mix vendors, and what does checkout do about
  it? (`oscar-marketplace` — documented hint: "ensure at checkout that a
  basket only contains lines from one partner"; Best Practice 1: split at
  order level, never per-line shipping)
- Decision: The basket is a **multi-chef holding container**, always
  displayed **grouped by chef** with per-group subtotal, delivery-fee
  estimate and a "Checkout" action (screen #20). **Checkout is scoped to
  one chef group at a time**: the selected partition is extracted into a
  checkout basket (lines moved, then frozen through submission as Oscar
  expects); the open basket keeps the other groups; abandon/payment
  failure merges the extraction back (Oscar merge semantics — strategy and
  offers re-applied). Every order is therefore **single-chef** — the
  documented hint holds at order level with no payment-splitting
  machinery. A chef group mixing live and pre-order lines (or two cook
  dates) requires a fulfilment-slot choice at checkout: one checkout
  serves one delivery moment. Guard: `HOMECHOW_MAX_CHEFS_PER_BASKET`
  (default 5 — cart hygiene only).
- Mechanism: `services/basket_rules.py` (`partition(basket)`, caps) called
  by the forked `BasketAddView` and the API; chef-scoped extraction +
  merge-back inside `services/checkout.SubmissionService`. Ladder rung: 4
  (fork + narrow view override) + service layer.
- Rejected:
  - Single-chef basket (v1 proposal) — rejected in review: the cart
    screens require multi-chef browsing/holding.
  - One payment → N orders via an `OrderGroup` split (v2 proposal) —
    rejected in review clarification; it added payment allocation,
    proportional fees and partial-group refund complexity nobody asked
    for.
  - Multi-vendor lines in one order — per-line shipping/fulfilment split
    is the documented anti-pattern ("shipping charges and fulfilment split
    incorrectly forever after").
- Test: `tests/integration/test_checkout_scoping.py` — extraction takes
  exactly the selected partition; abandon merge-back restores
  lines/offers/totals; remaining groups unaffected; mixed-slot group
  requires slot selection; `tests/unit/basket/test_rules.py` partition +
  cap truth tables.
- Fork-manifest impact: `basket` (view override), `checkout` (session/
  views/applicator — see ADR-009)
