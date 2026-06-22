# ADR-006: Order & line status pipelines

- Status: accepted (2026-06-13)
- Key question: What status pipeline mirrors the real fulfilment process?
  (Oscar: `howto/how_to_set_up_order_processing`;
  `oscar-order-processing` Recipe 1 — model the real process first,
  including unhappy paths; statuses are customer-visible copy)
- Decision: Order pipeline
  `Placed → Accepted → Preparing → Ready → Out for delivery → Delivered →
  Completed`, with terminals `Rejected`, `Cancelled`, `Failed`.
  Line pipeline mirrors it (kept even though every order is single-chef —
  ADR-007 — so future per-line workflows stay open).
  `OSCAR_ORDER_STATUS_CASCADE` maps each order status onto its lines
  (safe because one chef per order; the documented caveat that cascade
  ignores line-pipeline restrictions is accepted and noted).
  Every transition has a named trigger (plan §4 table): chef actions,
  PIN-handoff shipping events, customer/ops cancels, Celery timers
  (60s accept window, 24h auto-complete). **All transitions go through
  `set_status`/the forked `EventHandler`** — raw status writes are a
  defect.
- Mechanism: `OSCAR_*` pipeline settings + forked
  `order/processing.py::EventHandler`. Ladder rung: 1 (settings) for the
  pipeline; rung 4 for the handler.
- Rejected:
  - Oscar's default pipeline — doesn't match kitchen/dispatch reality;
    renaming statuses later means data migrations over live orders.
  - Encoding rider/dispatch states into order statuses — delivery has its
    own `DeliveryJob` state machine; the order pipeline only carries what
    the customer must see.
  - PATCHable status field in the API — mutations are action endpoints
    calling the EventHandler (`oscar-order-processing` DRF guidance).
- Test: `tests/unit/order/test_pipeline.py` (initial ∈ pipeline, terminals
  empty, cascade targets exist — settings sanity per `oscar-testing`);
  `tests/integration/test_lifecycle.py` (transition truth table; illegal
  transition rejected with 409).
- Fork-manifest impact: `order` (models unchanged; overrides
  `order.utils.OrderNumberGenerator`, `order.processing.EventHandler`)
