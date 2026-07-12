# Discussion 006 — Phase 1 Auth Implementation Decisions

## Status

Resolved after Claude review and Mahdi approval.

Formalized in:

```txt
04-Specs/auth-phase-1.md
```

## Locked Scope

```txt
Phone-number OTP through SMS
One application-issued JWT session cookie
No Google OAuth
No email login
No password login
No password recovery
No refresh token
No refresh-session table
No Redis
```

A user who loses or clears their session requests another OTP. OTP verification is the complete login and recovery mechanism for Phase 1.

---

# Accepted Decisions

## Identity

```txt
Primary and only Phase 1 identity anchor: normalizedPhone
Canonical format: E.164
Example: +989121234567
```

Rules:

- `normalizedPhone` is unique in PostgreSQL
- raw input is never used as an identity key
- no `user_identities` table
- no account linking or provider merging
- Phase 1 accepts Iranian mobile numbers only

## Session model

Use one JWT stored in one cookie:

```txt
JWT lifetime: long enough to cover the closed validation window
Suggested range: 14–30 days
Cookie: HttpOnly + Secure in production + SameSite=Lax + Path=/
Domain: omitted unless deployment requires it
```

Do not implement:

- access/refresh token split
- refresh endpoint
- refresh-token rotation
- token-family reuse detection
- persisted device sessions
- `replacedBySessionId`

## Revocation

Add to `User`:

```txt
sessionEpoch: Integer, required, default 0
```

Add to JWT claims:

```txt
sessionEpoch
```

Validation rule:

```txt
jwt.sessionEpoch == user.sessionEpoch
```

Behavior:

- normal logout clears the cookie on the current browser
- logout-all or forced revocation increments `User.sessionEpoch`
- incrementing the epoch invalidates all previously issued JWTs for that user
- per-device revocation is excluded from Phase 1

## JWT claims

Minimum claims:

```txt
sub: internal user id
iss: application issuer
iat
exp
jti
sessionEpoch
```

Do not include phone numbers, OTP data, profile details, or behavioral data in the JWT.

## Deployment and CSRF

Deployment remains same-origin:

```txt
https://app-domain/
https://app-domain/api/
```

Nginx serves the frontend and proxies `/api` to Spring Boot.

Phase 1 CSRF policy:

- `SameSite=Lax`
- state-changing endpoints accept JSON only
- validate `Origin` and/or `Referer` for unsafe methods
- no synchronizer-token or double-submit-token mechanism in Phase 1
- explicit CORS credentials/origin tests remain mandatory

## OTP persistence

OTP challenges are stored in PostgreSQL.

Minimum fields:

```txt
id
normalizedPhone
purpose
codeDigest
expiresAt
attemptCount
usedAt nullable
createdAt
```

Possible provider/debug fields remain optional:

```txt
providerMessageId
lastSentAt
requestIpHash
userAgentHash
```

## OTP purpose

Keep:

```txt
LOGIN
```

The purpose field remains future-safe, but the `wrong-purpose OTP` test is deferred until a second purpose exists.

## OTP hashing

Do not store plaintext OTP values.

Accepted Phase 1 approach:

```txt
HMAC-SHA-256(serverSecret, normalizedPhone + purpose + code)
```

A server-side secret/pepper is required because short numeric OTP values have low entropy and plain SHA-256 would be cheaply brute-forced after a database leak.

Do not use bcrypt or Argon2 for OTP challenges unless implementation evidence shows a concrete need.

## Phone normalization

Accept common Iranian input forms:

```txt
09121234567
989121234567
+989121234567
```

Normalize to:

```txt
+989121234567
```

Use a small Iran-specific normalizer rather than adding a general international phone-number dependency in Phase 1.

Frontend validation is feedback only. Backend normalization and validation own correctness.

---

# Transaction and Race-Condition Rules

## OTP verification transaction

The following must occur in one PostgreSQL transaction:

```txt
lock/load active OTP challenge
validate code, expiry, usage state, and attempt count
mark OTP as used
create or load User by normalizedPhone
commit
```

JWT creation and cookie writing occur after the transaction commits.

A crash after commit may require the user to request another OTP, but must not corrupt identity or OTP state.

## Duplicate-user protection

The database unique constraint is the final safety boundary:

```txt
UNIQUE(users.normalized_phone)
```

A naive `find then insert` check is not sufficient.

When concurrent verification requests race for one new phone number:

- exactly one transaction creates the user
- the losing transaction handles the unique-constraint conflict
- the losing path reloads the existing user instead of returning an internal error
- the system must never create two users for one normalized number

This race is distinct from consuming one OTP twice and requires its own integration test.

---

# Required Tests

## Backend integration tests

```txt
phone normalization produces one canonical identity
normalized phone uniqueness is enforced
OTP request stores no plaintext code
expired OTP is rejected
used OTP is rejected
maximum-attempt behavior follows Mahdi's configured rule
OTP verification creates a new user exactly once
concurrent verification cannot consume the same OTP twice
concurrent verification for one new phone creates exactly one User
unique-constraint race is handled without a 500 response
successful verification issues a valid JWT cookie
JWT sessionEpoch must match User.sessionEpoch
incrementing sessionEpoch invalidates older JWTs
logout clears the current cookie
production cookie attributes are correct
unsafe cross-origin mutation is rejected
```

Deferred:

```txt
wrong-purpose OTP test until a second purpose exists
refresh-token tests
per-device session revocation tests
```

## Frontend / E2E tests

```txt
request OTP
resend cooldown UI follows backend state
submit invalid OTP
submit expired OTP
successful new-user login enters Day-0
successful returning-user login enters the app
reload preserves a valid cookie session
logout clears the session
expired or revoked session returns the user to auth without losing server data
```

---

# Values Mahdi Will Define Before Implementation

```txt
OTP expiration duration
resend cooldown
maximum verification attempts
request rate limit
single-use invalidation behavior
OTP code length and format
JWT lifetime
SMS provider
provider timeout and retry policy
cookie name
logout-all product exposure
```

These values are implementation checkpoints, not unresolved architecture.

---

# Review Resolution

Accepted from Claude review:

- remove password recovery
- replace rotating refresh sessions with one JWT and `sessionEpoch`
- use HttpOnly cookies
- keep same-origin + SameSite=Lax + Origin/Referer checks
- store OTP challenges in PostgreSQL
- use an Iran-specific phone normalizer
- add explicit duplicate-user race handling
- remove the currently meaningless wrong-purpose test

GPT amendment accepted by Mahdi:

- OTP digest uses keyed HMAC/pepper, not raw SHA-256, because numeric OTPs are low entropy

This discussion is closed. Future changes belong in the formal auth spec or a new numbered discussion.