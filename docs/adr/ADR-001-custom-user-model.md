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

## Amendment (2026-06-20): alternate logins by phone or email

The original decision stands — **phone remains the account identity**
(`USERNAME_FIELD`, required, unique) and registration is phone-OTP, with
email optional. We additionally allow **alternate logins** for accounts that
opt in, without relaxing that invariant:

- **Password login** — `POST /auth/login {identifier, password}` where
  `identifier` is the phone (E.164) or the email. A user enables it by setting
  a usable password via `POST /auth/set-password {password, email?}`
  (mobile users start with an unusable password, so OTP-only accounts cannot
  password-login).
- **Email OTP login** — `/auth/otp/request` and `/auth/otp/verify` now accept
  an email `identifier`. An email code is only issued to an **existing**
  account (no email-only signup), and email verify logs in rather than
  creating an account. The OTP store (`PhoneOTP.phone`) was widened to 254
  chars to hold either identifier; `sent_via` records the channel.

Constraints kept: no email-only accounts (email verify on an unknown address
returns `no_account`); responses stay non-enumerating; `EmailBackend` still
serves staff dashboard login. Net change is additive — no new user-model
fields, one widening migration on `PhoneOTP`.
