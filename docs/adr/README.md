# HomeChow ADR register

Architecture decision records for the HomeChow backend, written to the
template in `oscar-architecture` (Code Examples). These files are the
audit trail the framework's upgrade and quarterly-review procedures
consume; they move into `homechow-backend/docs/adr/` in the first commit.
as revised in the review of 2026-06-12/13.

| ADR | Decision | Status |
|---|---|---|
| [ADR-001](ADR-001-custom-user-model.md) | Custom phone-first user model (day-one) | accepted |
| [ADR-002](ADR-002-database-postgis.md) | PostgreSQL + PostGIS, `ATOMIC_REQUESTS` | accepted |
| [ADR-003](ADR-003-api-stance.md) | Evaluate django-oscar-api; expect custom DRF layer | accepted — evaluation evidence due M0 |
| [ADR-004](ADR-004-product-classes.md) | One `dish` product class; attributes; standalone only | accepted |
| [ADR-005](ADR-005-currency-tax.md) | NGN, tax-known zero-itemized strategy | accepted |
| [ADR-006](ADR-006-order-pipelines.md) | HomeChow order/line status pipelines + EventHandler | accepted |
| [ADR-007](ADR-007-basket-partitioning.md) | Multi-chef basket, per-chef checkout | accepted (clarified in review) |
| [ADR-008](ADR-008-vendor-modelling.md) | Chef = Partner (undiverged) + ChefProfile; riders not partners | accepted |
| [ADR-009](ADR-009-payment-timing.md) | Instant charge at placement; wallet+Paystack; no COD | accepted |
| [ADR-010](ADR-010-back-office.md) | Oscar dashboard staff-only; chefs/riders API-only | accepted |
| [ADR-011](ADR-011-anonymous-checkout.md) | Anonymous checkout disabled | accepted |
| [ADR-012](ADR-012-search.md) | SearchGateway over Postgres FTS+geo; Solr deferred | accepted |
| [ADR-013](ADR-013-realtime.md) | Django Channels + polling fallback | accepted |
| [ADR-014](ADR-014-hidden-features.md) | Hide Oscar reviews/wishlists; domain replacements | accepted |

Process: new architecturally-significant decisions get the next number and
a PR touching this index. A superseded ADR is never deleted — its status
becomes `superseded by ADR-NNN`.
