# HomeChow backend — pre-implementation document set

Everything here is reviewed in this repo first, then seeds
`homechow-backend` in M0. Source of truth for what gets built:
the implementation plan; the rest derive from it.

| Document | What it is | Status |
|---|---|---|
| [backend-implementation-plan.md](backend-implementation-plan.md) | The module-by-module build plan (v3 — review decisions applied) | ✅ approved decisions in §11 |
| [adr/](adr/README.md) | ADR-001…014 — the day-one decision records (oscar-architecture template) | drafted; ADR-003 evidence due M0 |
| [fork-manifest.yaml](fork-manifest.yaml) | Customisation-debt inventory (6 forks); CI-enforced in the backend repo | drafted |
| [comms-register.yaml](comms-register.yaml) | Event-code → template-source → channels matrix (incl. push-only events) | drafted |
| [api-contract-v1.md](api-contract-v1.md) | The HTTP + WebSocket contract the three mobile apps build against | draft for review |
| [data-model-reference.md](data-model-reference.md) | ERD — Oscar core + domain entities, cross-module joins (Mermaid) | draft for review |
| [state-machines.md](state-machines.md) | Order / DeliveryJob / Drop / KYC / Withdrawal state machines + triggers (Mermaid) | draft for review |
| [backlog.md](backlog.md) | M0–M7 → epics → tickets with deps & sizes | draft for review |
| [integration-register.md](integration-register.md) | Providers, env vars, webhooks, **lead-time watch** | scaffold — ⚠ needs inputs |
| [nfr-security.md](nfr-security.md) | Perf targets, threat model, NDPR/privacy, reliability | scaffold — ⚠ needs inputs |

The set is complete. Two documents are scaffolds with clearly-marked ⚠
inputs (account owners, scale/SLO numbers, NDPR retention values) that need
the business or counsel — everything else is reviewable now.

**Act now (lead time):** start Paystack business verification + Transfers
approval and Termii sender-ID registration during M0 — see
[integration-register.md §1](integration-register.md).
