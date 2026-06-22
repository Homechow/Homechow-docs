# ADR-005: Currency & tax — NGN, tax-known, zero-itemized

- Status: accepted (2026-06-13)
- Key question: How is tax calculated and how are prices displayed?
  (Oscar: `topics/prices_and_availability`, `topics/key_questions`;
  `oscar-pricing-availability`)
- Decision: `OSCAR_DEFAULT_CURRENCY = 'NGN'`, formatted `₦#,##0.00`.
  The chef's drop price **is** the final consumer item price. The strategy
  returns `FixedPrice(excl_tax == incl_tax, tax=0, is_tax_known=True)` —
  a NoTax-style stance: tax is not itemized to customers at launch.
  Money serializes everywhere as
  `{"currency","excl_tax","incl_tax","is_tax_known"}` so a future VAT
  itemization (e.g. 7.5% on the platform service fee) is a strategy/
  surcharge change only — zero schema or contract impact.
- Mechanism: forked `partner` app — `Selector` + `HomeChowStrategy`
  (`oscar-pricing-availability` Recipe 1 shape). Ladder rung: 4
  (fork + class override).
- Rejected:
  - Deferred tax (`DeferredTax`/US pattern) — NG pricing is
    inclusive-final; deferred tax complicates every client for nothing.
  - Per-product tax fields — rung-5 schema debt; rates belong in
    strategy-side tables if ever needed (docs' own `FixedRateTax` caveat).
  - Tax maths in serializers/templates — would not hold across surfaces
    (architecture invariant #1).
- Test: `tests/unit/partner/test_strategy.py::test_ngn_tax_known_zero`,
  `::test_incl_equals_excl`;
  `tests/api/test_contracts.py::test_money_shape_pinned`
- Fork-manifest impact: `partner` (models unchanged; overrides
  `partner.strategy.Selector`, `partner.strategy.HomeChowStrategy`)
