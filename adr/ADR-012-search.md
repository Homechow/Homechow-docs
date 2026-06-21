# ADR-012: Search — SearchGateway over Postgres FTS + geo; Solr deferred

- Status: accepted (2026-06-13)
- Key question: What powers discovery (search, browse, "near you")?
  (Oscar: Haystack is a hard dependency; `oscar-search` Recipe 5 — the
  service seam for alternative backends is the documented deviation path)
- Decision: A `services/search.py::SearchGateway` seam owns ALL discovery
  queries: PostgreSQL full-text (dish title/description/tags, chef names)
  combined with PostGIS radius/zone filters and drop-state filters,
  returning ids hydrated through strategy-priced serializers
  (`oscar-drf-api` Recipe 6). Haystack stays wired with `SimpleEngine`
  purely to satisfy Oscar's dependency (its web search views are unused —
  the storefront is headless). Solr (or another engine) becomes a gateway
  implementation swap when catalogue scale or relevance tuning demands
  it — no API contract change.
- Mechanism: service seam + Haystack `SimpleEngine` setting. Ladder rung:
  service layer (no fork); rung 1 for the Haystack wiring.
- Rejected:
  - Solr from day one — the install/schema/reindex runbook
    (`oscar-search` Recipe 1) is real ops weight; a city-scale dish
    catalogue doesn't need it at launch.
  - Querying Haystack from the API — the gateway keeps geo + drop-state +
    text in one query planner and makes the backend swappable.
  - Elasticsearch managed service at MVP — cost + an extra moving part;
    revisit at the Solr decision point.
- Test: gateway contract tests (text match, geo radius, drop-state
  filters, pagination) with `assertNumQueries` budget; API search endpoint
  pins its payload; a degraded-mode test (gateway falls back to plain DB
  filters).
- Fork-manifest impact: none (`search` unforked; SimpleEngine via settings)
