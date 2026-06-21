# HomeChow backend pre-implementation document set

Everything here is reviewed in this repo first, then seeds
`homechow-backend`  Source of truth for what gets built:
the implementation plan; the rest derive from it.

| Document | What it is | Status |
|---|---|---|
| [backend-implementation-plan.md](backend-implementation-plan.md) | The module-by-module build plan (v3 — review decisions applied) | ✅ approved decisions in §11 |
| [adr/](adr/README.md) | the day-one decision records (oscar-architecture template) | drafted; ADR-003 evidence due M0 |
| [comms-register.yaml](comms-register.yaml) | Event-code → template-source → channels matrix (incl. push-only events) | drafted |
| [api-contract-v1.md](api-contract-v1.md) | The HTTP + WebSocket contract the three mobile apps build against | draft for review |
| [data-model-reference.md](data-model-reference.md) | ERD — Oscar core + domain entities, cross-module joins (Mermaid) | draft for review |
| [state-machines.md](state-machines.md) | Order / DeliveryJob / Drop / KYC / Withdrawal state machines + triggers (Mermaid) | draft for review |
| [integration-register.md](integration-register.md) | Providers, env vars, webhooks, **lead-time watch** | scaffold — ⚠ needs inputs |
| [nfr-security.md](nfr-security.md) | Perf targets, threat model, NDPR/privacy, reliability | scaffold — ⚠ needs inputs |

