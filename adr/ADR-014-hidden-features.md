# ADR-014: Hidden Oscar features — product reviews & wishlists off

- Status: accepted (2026-06-13)
- Key question: Which stock Oscar features don't fit the domain?
  (Oscar: `OSCAR_HIDDEN_FEATURES`; `oscar-architecture` ladder rung 6 —
  feature hiding without forks)
- Decision: `OSCAR_HIDDEN_FEATURES = ['reviews', 'wishlists']`.
  HomeChow reviews target **chefs and riders per completed order**
  (`domain/reviews/`: `ChefReview` order-OneToOne with reply support,
  `RiderReview` job-OneToOne) — a different contract from Oscar's
  per-product `catalogue.reviews` (anonymous-allowed, product-scoped).
  "Wishlist" intent is covered by chef **follows** + **DishAlert**
  ("notify me on next drop") in `domain/engagement/` — drop-based push,
  not product-page email. Hidden features simply have no endpoints
  (contract-level feature gating per the DRF blueprint).
- Mechanism: setting (rung 6 feature hiding) + domain apps for the
  replacements. No `catalogue` fork needed (forking `catalogue.reviews`
  would have required forking `catalogue` — the parent rule).
- Rejected:
  - Bending `catalogue.reviews` into chef reviews — reviews would attach
    to dish products, not chefs/orders; moderation, reply and
    one-review-per-order semantics all fight the stock model.
  - Oscar `ProductAlert` for drop alerts — stockrecord-update email
    alerts; ours are publish-time push fan-outs to followers, a different
    trigger and channel.
  - Keeping wishlists alongside follows — two overlapping "save this"
    concepts confuse the product.
- Test: settings sanity pins the hidden list; API smoke asserts no
  `/reviews|wishlists` Oscar routes are exposed; `domain/reviews` tests
  own the replacement behaviour (one review per delivered order, reply
  flow, aggregate denormalization).
- Fork-manifest impact: none
