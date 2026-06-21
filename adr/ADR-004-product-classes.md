# ADR-004: Catalogue — one `dish` product class, attributes, standalone only

- Status: accepted (2026-06-13)
- Key question: What product types exist and how are they modelled?
  (Oscar: `topics/modelling_your_catalogue`; `oscar-catalogue` Recipe 1)
- Decision: Exactly one product class, `dish`
  (`track_stock=True`, `requires_shipping=True`), created by data
  migration. Dish facts are **attributes**: `dietary` (multi_option:
  Spicy, Halal, Gluten-Free, Vegan, …), `cuisine` (multi_option),
  `portion_size` (text), `prep_minutes` (integer), `ingredients`
  (richtext). Categories (Nigerian, Snacks, Pastries, …) are seeded with
  `create_from_breadcrumbs` and stay **navigational only**; any business
  grouping (offers) uses Ranges. **All products are standalone** — no
  parent/child variants: the permission-scoped vendor mechanics document
  parent/child as unsupported, and "variants" of a dish (portion sizes)
  are simply separate dishes with their own drops/prices.
- Mechanism: `domain/bootstrap/` data migration for class + attribute
  schema + option groups + categories. Ladder rung: attributes (data
  ladder rung 1 — no model change, no fork).
- Rejected:
  - A product class per cuisine/category — classes exist for behavioural
    splits (stock/shipping/attribute set), not merchandising
    (`oscar-catalogue` Best Practice 1).
  - Parent/child variant families — documented marketplace limitation;
    needless complexity for dishes.
  - `prep_minutes`/`portion_size` as Product model fields — the data
    ladder says attributes suffice; forking `catalogue` starts the
    migration-debt clock for nothing.
- Test: `tests/unit/catalogue/test_schema.py::test_dish_class_and_attributes`
  (post-migration assertions per `oscar-catalogue` Testing Guidance);
  attribute round-trip test; `filter_by_attributes(dietary='Halal')`.
- Fork-manifest impact: none (`catalogue` unforked — recorded in the
  manifest notes)
