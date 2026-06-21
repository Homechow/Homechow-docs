# ADR-009: Payment — instant charge at placement; wallet+Paystack; no COD

- Status: accepted (2026-06-13) — instant pre-order charge and no-COD
  confirmed in review
- Key question: Which payment sources, in what combinations, charged when?
  (Oscar: `howto/how_to_integrate_payment`, `topics/key_questions`;
  `oscar-payment`)
- Decision: **One-phase charge at placement** (`amount_debited`) for live
  orders **and pre-orders alike** — Paystack's NG card rails are
  capture-style; long auth holds are unreliable, and refunds inside the
  pre-order cancel window handle the unhappy paths. One charge per
  checkout = one single-chef order (ADR-007). Sources: `wallet`
  (ledger debit), `paystack-card` (saved authorization), `paystack-transfer`
  / new card (RedirectRequired → `payment_action`). Split payment
  wallet-then-card per `oscar-payment` Recipe 5 including the unwind
  (re-credit wallet if the card leg fails). The four documented payment
  exceptions are the complete error vocabulary; gateway codes map through
  ONE table in `integrations/payments/paystack/` (Recipe 3). Idempotency:
  `Idempotency-Key` on submit + `pay-{order_number}-{attempt}` gateway
  references. **Pay-on-delivery is out of scope**: no COD source type, no
  rider cash ledger.
- Mechanism: forked `checkout` — `PaymentDetailsView.handle_payment` +
  `SubmissionService`; Paystack client behind `integrations/`. Ladder
  rung: 4 (fork + narrowest overrides).
- Rejected:
  - Pre-auth then capture-on-delivery — weak NG gateway support for holds;
    adds settle-on-ship complexity with no customer benefit here.
  - COD at launch — rider cash reconciliation, fraud surface, and a
    second settlement pipeline (review decision: out).
  - Catch-all exception handling around payment — documented anti-pattern;
    collapses the retryable/non-retryable UX distinction.
- Test: `tests/unit/payments/test_exception_mapping.py` (gateway code →
  Oscar exception table); `tests/integration/test_payment_paths.py`
  (success persists Source/Transaction/event atomically; each exception
  path restores an editable basket; split unwind; webhook replay
  idempotency); API exception→HTTP contract test.
- Fork-manifest impact: `checkout` (overrides
  `checkout.session.CheckoutSessionMixin`,
  `checkout.views.PaymentDetailsView`,
  `checkout.applicator.SurchargeApplicator`)
