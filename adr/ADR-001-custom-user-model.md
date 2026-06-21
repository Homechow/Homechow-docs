# ADR-001: Custom user model (phone-first identity)

- Status: accepted (2026-06-13)
- Key question: Who is the user and how do they authenticate? (Oscar:
  `howto/use_a_custom_user_model` — the documented day-one timing trap)
- Decision: Custom `users.User` subclassing
  `oscar.apps.customer.abstract_models.AbstractUser`, decided **before the
  first `migrate`**. Fields: `phone` (E.164, unique, required,
  `USERNAME_FIELD`) + `email` (unique, nullable — staff dashboard login,
  receipts). Customer/chef/rider authentication is OTP→JWT against phone;
  staff authenticate email+password via Oscar's `EmailBackend`
  (listed before `ModelBackend`). One user may hold several roles
  (customer who is also a chef); roles derive from related profiles, never
  from user-model flags that duplicate them.
- Mechanism: `AUTH_USER_MODEL = 'users.User'` in `homechow/domain/users/`;
  Oscar's account machinery auto-adapts to the model. Ladder rung: model
  subclass (the sanctioned day-one decision — the `customer` app itself is
  NOT forked).
- Rejected:
  - Starting on `auth.User` "temporarily" — documented trap: switching
    later requires manual table renames over live data.
  - Email-as-username — the three mobile audiences authenticate by phone
    OTP per the prototype; email stays unique-when-present so
    `EmailBackend` keeps working for staff.
  - Storing commerce profile data (ratings, KYC state) on the user model —
    profile data lives in related models (`oscar-customer` anti-pattern:
    user-model migrations are the most painful kind).
- Test: `tests/unit/users/test_user_model.py::test_phone_unique_e164`,
  `::test_email_unique_nullable`;
  `tests/integration/test_auth.py::test_otp_to_jwt_roundtrip`,
  `::test_staff_email_backend_login`
- Fork-manifest impact: none (`customer` unforked; `users` is a domain app)
