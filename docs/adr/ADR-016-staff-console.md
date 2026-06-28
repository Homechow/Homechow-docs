# ADR-016: Branded staff console beside Oscar's dashboard (amends ADR-010)

- Status: accepted (2026-06-23) — amends ADR-010
- Key question: ADR-010 made Oscar's dashboard the *only* staff back-office
  and explicitly rejected "a separate ops SPA." With the product now wanting
  a back-office that **looks and feels like the HomeChow mobile app** (and is
  more interactive than full-page Oscar forms), is that still the right call?
- Decision: Add a **standalone, branded staff console** at **`/staff/`** — a
  plain Django app (`homechow/staff/`) rendering **Bootstrap 5 + htmx + Alpine**
  templates in the app's own design language (Hanken Grotesk; warm-cream canvas;
  green-primary with terracotta/blue/amber/clay accents; rounded cards, pill
  badges, status timelines — sampled from `HomeChow UI/`). It becomes the
  **primary staff surface**; **Oscar's `/dashboard/` stays mounted** for deep
  Oscar-native CRUD (catalogue, offers, reports). This **amends ADR-010** (the
  "Oscar dashboard is the *only* back-office" / "no separate ops surface"
  clause); everything else in ADR-010 stands.
  - **Reuse, don't reinvent.** Views are thin: read-only KPI/list queries hit
    the domain models directly (the convention the ops sub-apps already use);
    every **mutation goes through `services/`** (`kyc`, `dispatch`, `payouts`,
    `support`, `payments.refund_order`, …). No business rule is re-implemented.
  - **No second API.** There is no staff/admin DRF surface, and we don't add
    one: the panel is server-rendered, htmx posts back to **Django view**
    endpoints that return HTML partials. The JSON API stays exclusively for the
    mobile clients (customer/chef/rider). A JSON API earns its keep when a
    *separate networked client* needs the operation — false for internal staff.
  - **Staff-only.** `StaffRequiredMixin` (`is_staff`): unauthenticated → branded
    `/staff/login` (phone or email + password, Django session auth);
    authenticated non-staff → 403. Chefs/riders remain API-only (ADR-010).
  - **Layering.** `homechow.staff` joins `homechow.api` as a presentation layer
    in the import-linter contract — domain/services may never import it.
  - The existing `kyc`/`ops` `OscarDashboardConfig` sub-apps are **migrated**
    into the console (same `services` calls, same `is_staff`/`by_user` audit)
    and retired once the branded screens reach parity.
- Mechanism: new standalone Django app + `/staff/` URL mount + CDN assets
  (Bootstrap/htmx/Alpine/Hanken Grotesk). No Oscar fork; ladder rung n/a
  (presentation app beside Oscar, like the API).
- Rejected:
  - **Keep Oscar-only** (status quo) — can't carry the app's brand/interactivity;
    stuck on Oscar's Bootstrap-4 shell.
  - **Replace Oscar's dashboard wholesale** — needless rebuild of catalogue/
    offers/reports CRUD that Oscar gives for free.
  - **Staff JSON API + JS SPA** — duplicates, over HTTP, the one-line service
    calls server-rendered views already make; adds an `IsStaff` API, JWT-in-
    browser, serializers and schema churn for no offline/mobile need.
  - **Vendor (chef/rider) web logins** — out of scope; ADR-010's API-only stance
    holds (the permission plumbing stays warm for a later flip).
- Test: staff access matrix (anonymous / customer / staff × routes); login by
  phone/email, is_staff gate, safe-`next` handling, logout; Overview KPI
  aggregates. Per-phase: render smoke + service-call assertions for each
  mutation screen. Not DRF → no OpenAPI snapshot impact.
- Fork-manifest impact: none (standalone app; Oscar apps unforked; ADR-010's
  existing `dashboard.orders` fork unchanged).
