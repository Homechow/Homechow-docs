# ADR-015: Image handling — Cloudflare Images (direct upload) + R2 for KYC

- Status: accepted (2026-06-22)
- Key question: Where do user-supplied images (chef avatars/covers, dish
  photos, rider headshots, pickup/delivery evidence) and KYC documents
  live, who uploads the bytes, and how does the API serve sized,
  access-controlled variants? (No Oscar coverage — Oscar stores
  `ProductImage` files on the configured Django storage; HomeChow is
  image-heavy and mobile-first, so this is project infrastructure beside
  the catalogue.)
- Decision: **Cloudflare Images** for user photos, **direct creator
  upload**: the client asks the backend for a one-time upload URL
  (`POST /api/v1/uploads/image {purpose}` → `{id, upload_url}`), PUTs the
  raw bytes straight to Cloudflare, then sends the returned **image id**
  back to whichever endpoint owns the field (e.g.
  `PATCH /chef/profile {avatar: id}`). The bytes never transit Django — no
  multipart bottleneck on the web dynos, no media volume to back up.
  Cloudflare resizes on the fly via named **variants** (`thumb`, `card`,
  `cover`, `public`); the API returns delivery URLs
  (`https://imagedelivery.net/<hash>/<id>/<variant>`). Pickup and delivery
  **evidence** photos are private — uploaded with `requireSignedURLs` and
  served as short-lived **signed URLs** (only staff/parties to the order
  see them).
  - **Image fields keep the Django `ImageField` API.** A custom
    `CloudflareImagesStorage` (infrastructure/storages.py) stores the CF
    *id* as the field's `name` and builds the delivery URL in `.url`. The
    `storage=` is a **callable selector** (`select_image_storage` /
    `select_private_image_storage`), so migrations record the callable, the
    backend is chosen at runtime from settings, and flipping
    `HOMECHOW_USE_CLOUDFLARE_IMAGES` needs **no new migration**. Dev/test
    fall back to local `FileSystemStorage`.
  - **KYC documents go to Cloudflare R2** (S3-compatible, private bucket)
    via `django-storages` + `boto3`, behind `select_doc_storage`
    (`HOMECHOW_USE_R2`). KYC is sensitive and retained for compliance, so
    it is isolated from the public-image path and served only by signed
    URLs. boto3/django-storages are imported lazily — absent in dev/test.
  - **One integration seam** (`integrations/cloudflare/images.py`) with an
    in-process **fake** (returns deterministic ids, no network) when no
    token is set — the paystack/maps pattern. `services/images.py` owns the
    app verbs: `create_upload`, `attach` (swap id in, delete the one it
    replaces so Cloudflare accrues no orphans), `variant_url`.
- Mechanism: new `api/v1/uploads` sub-app (tag **Uploads**); image fields
  on `ChefProfile.avatar/cover_photo`, `ChefDish.photo`, `RiderProfile.photo`
  (public) and `DeliveryJob.pickup_photo/proof_photo` (private);
  `ChefApplicationDocument.file` / `RiderApplicationDocument.file` on R2.
  Ladder rung: n/a (infrastructure beside Oscar; catalogue stays unforked —
  ADR-004 — the dish photo hangs off the related `ChefDish`, not `Product`).
- Rejected:
  - **Server-proxied uploads** (multipart to Django → forward to storage) —
    doubles bandwidth and ties up web workers on large mobile photos; the
    whole point of Cloudflare direct upload is to skip the origin.
  - **S3/CloudFront** for images — works, but we'd own the resize pipeline
    (Lambda@Edge / thumbor); Cloudflare Images bundles named-variant
    resizing, signed URLs and a CDN for a flat per-image price.
  - **Local/volume media in production** — no CDN, a stateful volume to
    back up and migrate, and slow first-byte to Lagos.
  - One bucket for images **and** KYC — conflates public CDN content with
    retained compliance documents; different access models and lifecycles.
- Test: `tests/integration/test_images.py` — upload ticket (auth, purpose
  validation, signed-flag for private purposes), `attach` (id set +
  orphan delete on replace), `variant_url` (local fallback vs imagedelivery
  URL; `full`→`public` mapping), signed-URL signing, and the wired
  chef-avatar / dish-photo / rider-photo / pickup-photo endpoints — all
  against the offline fake.
- Fork-manifest impact: none (catalogue, partner, order apps unforked;
  storage is additive via `storage=` callables and new fields).
