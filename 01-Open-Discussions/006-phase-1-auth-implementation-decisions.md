# Discussion 006 — Phase 1 Auth Implementation Decisions

## Status

Waiting for Claude review.

## Context

The Phase 1 product scope, technical foundation, implementation stack, and data model are now defined.

Relevant source documents:

- [[02-Decisions/ADR-002-phase-1-technical-foundation]]
- [[04-Specs/phase-1-implementation-stack]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/day-0-onboarding]]

Phase 1 authentication is intentionally limited to:

```txt
Phone-number OTP through SMS
JWT-based application sessions
No Google OAuth
No email login
No social login
```

The remaining work is not to reopen the authentication-method decision. It is to define the implementation rules needed before backend and frontend scaffolding begins.

---

# Locked Decisions

These decisions are not open for review unless they create a direct implementation contradiction.

## Identity

```txt
Primary and only Phase 1 identity anchor: normalized phone number
```

Rules:

- each user has one unique normalized phone number
- no polymorphic `user_identities` table in Phase 1
- no provider linking or account merging
- Iranian phone numbers must be stored in one canonical format
- raw user input must never be used as the uniqueness key

Proposed canonical storage format:

```txt
E.164
Example: +989121234567
```

## Authentication method

```txt
Request OTP
→ receive SMS
→ verify OTP
→ create or authenticate user
→ issue application session
```

A new user enters Day-0 onboarding after successful OTP verification.

An existing user enters the application after successful OTP verification.

## OTP provider boundary

Provider-specific SMS code stays behind an application-owned interface:

```java
public interface OtpDeliveryGateway {
    void send(String normalizedPhoneNumber, String code);
}
```

Controllers and domain/application services must not depend directly on a specific Iranian SMS provider SDK.

## Deployment boundary

Recommended deployment remains same-origin:

```txt
https://app-domain/
https://app-domain/api/
```

Nginx serves the frontend and proxies `/api` to Spring Boot.

This is intended to reduce cookie, CORS, and browser-policy complexity.

---

# Proposed Session Design

## Recommended approach

Use:

```txt
Short-lived access JWT
Long-lived refresh session
HttpOnly Secure cookies
```

Proposed cookie split:

```txt
access_token
refresh_token
```

Proposed responsibilities:

- access token authenticates normal API requests
- refresh token creates a new access token
- refresh sessions are persisted server-side so logout and revocation are possible
- raw refresh tokens are never stored in the database
- only a hash of the refresh token is persisted

## JWT claims

Minimum access-token claims:

```txt
sub: internal user id
iss: Adaptive Planner
iat
exp
jti
```

Do not include phone numbers, OTP data, profile details, or private behavioral data in JWT claims.

## Proposed cookie policy

```txt
HttpOnly: true
Secure: true in production
SameSite: Lax
Path: /
Domain: omitted unless deployment requires it
```

The refresh cookie may use a narrower path if the API structure makes it practical, for example:

```txt
Path: /api/v1/auth/refresh
```

## CSRF

Because authentication is cookie-based, CSRF behavior must be explicitly decided.

Proposed Phase 1 approach:

- same-origin frontend and API
- `SameSite=Lax`
- state-changing endpoints accept only JSON
- validate `Origin` and/or `Referer` for unsafe methods
- add a CSRF token only if the final browser/session design requires stronger protection

This point needs explicit Claude/backend review. `HttpOnly` protects tokens from JavaScript access, but it does not by itself solve CSRF.

---

# Proposed OTP Flow

## Request OTP

Proposed endpoint:

```txt
POST /api/v1/auth/otp/request
```

Request:

```json
{
  "phoneNumber": "09121234567",
  "purpose": "LOGIN"
}
```

The backend:

1. normalizes the phone number
2. applies request limits
3. generates an OTP
4. stores only a secure hash of the OTP
5. stores its purpose and expiry
6. sends the OTP through `OtpDeliveryGateway`
7. returns a generic success response

The response must not reveal whether the phone number already belongs to an account.

## Verify OTP

Proposed endpoint:

```txt
POST /api/v1/auth/otp/verify
```

Request:

```json
{
  "phoneNumber": "09121234567",
  "code": "123456",
  "purpose": "LOGIN"
}
```

The backend:

1. normalizes the phone number
2. loads the active OTP challenge
3. checks purpose, expiry, usage state, and attempt count
4. compares the submitted code against the stored hash
5. marks the OTP as used atomically
6. creates the user when no user exists for the normalized phone
7. creates the application session
8. returns the authenticated user/session response

## OTP storage

Proposed minimum OTP challenge fields:

```txt
id
normalizedPhone
purpose
codeHash
expiresAt
attemptCount
usedAt nullable
createdAt
```

Possible additional fields if required:

```txt
requestIpHash
userAgentHash
providerMessageId
lastSentAt
```

## OTP purpose

The OTP must be bound to a purpose so one code cannot be reused across unrelated flows.

Initial proposed enum:

```txt
LOGIN
```

---

# Password-Recovery Contradiction to Resolve

Earlier discussions mentioned:

```txt
Password recovery/reset through phone OTP
```

The current Phase 1 stack, however, defines OTP-only authentication and no password login.

If users do not have passwords, there is no password to forget or reset.

This discussion asks Claude to explicitly review this contradiction.

## GPT recommendation

For Phase 1:

```txt
No password field
No password login
No forgot-password flow
OTP verification is the complete login/recovery mechanism
```

A user who loses their session simply requests another OTP and logs in again.

Do not keep a fake "forgot password" concept merely because familiar auth screens usually contain one.

If Mahdi still wants password login later, that should be a separate post-Phase-1 decision with explicit password hashing, recovery, and account-security rules.

---

# Decisions Mahdi Will Define Before Implementation

The following values are intentionally not decided here. They must be filled before the auth module is implemented:

```txt
OTP expiration duration
Resend cooldown
Maximum verification attempts
Request rate limit
Single-use OTP invalidation behavior
```

Additional values that should be decided at the same implementation checkpoint:

```txt
OTP code length and format
Access-token lifetime
Refresh-token lifetime
Maximum active sessions per user
Refresh-token rotation behavior
Logout-all-devices behavior
SMS provider
Provider timeout and retry policy
```

This section is a reminder list, not a request for Claude to choose product policy on Mahdi's behalf.

---

# Refresh Session Proposal

Proposed server-side refresh-session fields:

```txt
id
userId
tokenHash
issuedAt
expiresAt
revokedAt nullable
replacedBySessionId nullable
lastUsedAt nullable
createdIpHash nullable
userAgentHash nullable
```

Recommended behavior:

- rotate refresh token on every successful refresh
- revoke the previous refresh session during rotation
- reject reuse of an already-rotated token
- logout revokes the current refresh session
- logout-all revokes every active refresh session for the user
- access JWTs remain short-lived and are not stored server-side

Claude should review whether this is appropriate for Phase 1 or unnecessarily heavy for a closed 10–20 tester validation.

---

# Phone Normalization Proposal

Accept common Iranian input forms:

```txt
09121234567
989121234567
+989121234567
```

Normalize all valid values to:

```txt
+989121234567
```

Reject:

- invalid lengths
- invalid Iranian mobile prefixes
- letters or malformed characters after reasonable sanitization
- non-Iranian numbers during the Iran-only Phase 1 test unless scope changes

Frontend validation is for immediate feedback only. Backend normalization and validation own correctness.

Claude should review whether a dedicated phone-number library is justified or whether an Iran-specific normalizer is simpler and safer for Phase 1.

---

# Security and Privacy Rules

The implementation must not:

- log OTP values
- log access or refresh tokens
- store OTP codes in plaintext
- store raw refresh tokens
- reveal whether a phone number is registered during OTP request
- trust frontend rate limiting
- use phone number as the JWT subject
- return internal provider errors directly to the client

The implementation should:

- use cryptographically secure random OTP generation
- compare OTP hashes safely
- make OTP consumption atomic
- keep provider credentials in environment secrets
- record authentication failures without sensitive values
- provide generic user-facing errors while preserving internal diagnostics

---

# Required Auth Tests

## Backend integration tests

```txt
phone normalization produces one canonical identity
normalized phone uniqueness is enforced
OTP request stores only a hash
expired OTP is rejected
used OTP is rejected
wrong-purpose OTP is rejected
maximum-attempt behavior follows Mahdi's configured rule
OTP verify creates a new user exactly once
concurrent verification cannot consume the same OTP twice
successful verification creates a valid session
refresh rotation revokes the previous refresh session
logout revokes the current refresh session
cookie attributes are correct in production profile
unsafe cross-origin request is rejected according to the final CSRF policy
```

## Frontend / E2E tests

```txt
request OTP
resend cooldown UI follows configured backend state
submit invalid OTP
submit expired OTP
successful new-user login enters Day-0
successful returning-user login enters the app
refresh restores the session
logout clears the session
expired session returns user to auth without losing server data
```

---

# Questions for Claude

1. Do you agree that OTP-only authentication means the password-recovery concept should be removed entirely from Phase 1?
2. Is the proposed access-JWT plus persisted rotating refresh-session model appropriate for Phase 1, or is there a simpler design that still supports secure logout and revocation?
3. Should access and refresh tokens both use HttpOnly cookies, or should only the refresh token use a cookie while the access token remains in memory?
4. Is `SameSite=Lax` plus same-origin deployment and Origin/Referer validation sufficient for Phase 1, or should a synchronizer/double-submit CSRF token be mandatory?
5. Should the refresh cookie use `Path=/` or a narrow refresh endpoint path?
6. Are the proposed refresh-session fields minimal and sufficient?
7. Is hashing OTP values and refresh tokens the correct storage rule for this scope?
8. Should OTP challenges be stored in PostgreSQL, or is there any compelling Phase 1 reason to introduce Redis? GPT position: PostgreSQL only; no Redis.
9. Is an Iran-specific phone normalizer preferable to adding a general international phone-number dependency?
10. Are there any missing transaction boundaries, especially around OTP consumption, user creation, and session creation?
11. Which auth tests are truly mandatory before the 10–20 person validation, and which can be deferred?
12. Are any proposed controls overengineered for Phase 1?

---

# Expected Outcome

After Claude review and Mahdi's final decisions, convert this discussion into a bounded auth implementation spec that owns:

```txt
phone normalization
OTP request and verification contracts
OTP persistence
JWT claims
refresh-session persistence
cookie policy
CSRF policy
logout and revocation
required auth tests
implementation checklist
```

This discussion should not reopen Google OAuth, email auth, password login, or multiple identity providers.
