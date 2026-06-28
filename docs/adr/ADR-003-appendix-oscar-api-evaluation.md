# ADR-003 Appendix — django-oscar-api evaluation (M0)

The approved stance (`oscar-drf-api` Recipe 1) requires evaluating
`django-oscar-api` before building a custom layer, and recording the
evidence either way. This is that record.

## Compatibility check (M0, 2026-06-13)

- Latest release: **django-oscar-api 3.3.0**, `Requires-Python >=3.7`,
  `Requires-Dist: django-oscar >=3.2`, `Django >=3.2`.
- Finding: the dependency pin has **no upper bound**, so pip would install
  it beside our Oscar 4.0 pin — but 3.3.0 was cut against the Oscar **3.x**
  line and does not advertise a 4.0-tested support matrix. This is exactly
  the documented "extension packages lag releases" risk
  (`oscar-payment`/`oscar-upgrade-strategy`): a permissive pin is not the
  same as a tested matrix.

## Requirements-matrix score (gap list)

| Requirement | django-oscar-api 3.3.0 | Fit |
|---|---|---|
| Auth for mobile | session-centric; header/token session support, no first-class **phone-OTP → JWT** | ✗ |
| Audiences | one storefront API; we need **customer / chef / rider** prefixes with distinct permission models | ✗ |
| Catalogue semantics | product/category endpoints; no concept of **drops** (time-boxed, stock-capped, live/pre-order availability) | ✗ |
| Checkout | mirrors Oscar placement; no **chef-scoped checkout extraction** (ADR-007) or **Paystack payment-action** contract out of the box | ✗ |
| Realtime | none — we need **WebSockets** for tracking/gig-offers/drop tickers | ✗ |
| Strategy-priced serializers | yes (respects forked apps via dynamic loading) | ✓ |
| Oscar 4.0 support matrix | not advertised | ✗ (upgrade gate risk) |

## Decision

**Build the custom DRF layer in `homechow/api/v1/`** (the expected outcome
in ADR-003), honouring the four invariants directly. The gaps above are
structural, not cosmetic — adopting the package would mean fighting its
session/auth model and endpoint shapes at every audience-specific route,
while still owning drops, chef-scoped checkout, payment-action and
WebSockets ourselves.

Door left open (per stance): re-check the package's supported-Oscar matrix
at each Oscar upgrade; if a 4.x-tested release with JWT and extensible
serializers lands, reconsider a hybrid (adopt its storefront basket/catalogue
endpoints beside our custom audiences). Tracked as open item O2.
