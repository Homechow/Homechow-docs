# ADR-010: Back-office — Oscar dashboard, staff-only; chefs/riders are API-only

- Status: accepted (2026-06-13)
- Key question: Who manages the store, with what tool?
  (Oscar: `ref/apps/dashboard` — the dashboard fully replaces Django
  admin; admin is unsupported for store management; `oscar-dashboard`)
- Decision: The Oscar dashboard is the **only** staff back-office; Django
  admin stays debug-only behind staff IP/VPN. New HomeChow staff modules
  are **`OscarDashboardConfig` sub-apps** (`dashboard_apps/`: chef/rider
  KYC queues, dispatch monitor, payout runs, support desk, zones,
  app-config) — zero extra Oscar forks. Chefs and riders get **no
  dashboard logins at MVP** — their surface is the mobile API — but the
  permission-based plumbing is kept warm: `Partner.users` is populated
  (ADR-008), `services/scoping.py` is the single queryset source for chef
  API *and* dashboard, and the forked `dashboard.orders` view already
  filters `get_order_lines()`. Granting a power-chef web access later is
  a group/permission flip, not a build.
- Mechanism: `OSCAR_DASHBOARD_NAVIGATION` setting + new sub-apps
  (`oscar-dashboard` Recipes 1/4) + one narrow `dashboard.orders` fork
  (Recipe 3). Ladder rungs: 1 (nav settings), 4 (orders view), plus
  upgrade-inert new sub-apps.
- Rejected:
  - Django admin as back-office — documented as unsupported.
  - Vendor dashboard logins at MVP — duplicates the chef app surface and
    triggers the documented scoped-dashboard limitations for no MVP gain.
  - A separate ops SPA — the server-rendered dashboard is free,
    permission-aware, and the skills own its extension points.
- Test: dashboard scoping matrix (staff / chef-with-permission /
  chef-without / anonymous × own/other/mixed resources) per
  `oscar-dashboard` Testing — run even though chef logins are off, so the
  flip stays safe; nav `url_name` reversal test; smoke: `/dashboard/`
  requires staff.
- Fork-manifest impact: `dashboard` parent + `dashboard.orders`
  (coupling rule: sub-app forks require the parent)
