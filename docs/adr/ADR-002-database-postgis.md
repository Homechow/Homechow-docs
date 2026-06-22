# ADR-002: Database — PostgreSQL + PostGIS

- Status: accepted (2026-06-13)
- Key question: What database and transaction posture?
  (Oscar: `ref/platform_database_support` — PostgreSQL is the only
  officially supported production database; MySQL/Windows unsupported)
- Decision: PostgreSQL 16 with the PostGIS extension, one primary database
  for commerce + geo. `ATOMIC_REQUESTS = True` from day one (checkout
  correctness quietly depends on transactional requests —
  `oscar-project-scaffolding`). Geo columns (`Address.location`,
  `ChefProfile.kitchen_location`, `ServiceZone.polygon`, job points) use
  `django.contrib.gis`. **Live rider positions do not hit the DB** — they
  live in Redis GEO with sampled audit rows only.
- Mechanism: settings + `django.contrib.gis` backend. Ladder rung: 1
  (configuration).
- Rejected:
  - MySQL / SQLite in production — explicitly unsupported by Oscar.
  - A separate geo datastore at MVP — operational overhead with no need at
    launch scale; the Redis GEO hot path already keeps location churn off
    Postgres.
  - Skipping PostGIS and using haversine on lat/lng floats — zone
    polygons (out-of-zone screens, tariffs, heatmaps) need real geometry.
- Test: `tests/conformance/test_settings.py::test_atomic_requests`,
  `::test_postgis_engine`; CI boot test runs `migrate --check` against
  PostGIS.
- Fork-manifest impact: none
