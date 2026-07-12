# ADR-002 — Phase 1 Technical Foundation

## Status

Accepted for Phase 1.

## Purpose

Record the minimum shared technical assumptions needed before finalizing the Phase 1 Data Model and implementation plan.

This ADR is intentionally bounded. Detailed frameworks and package choices are owned by [[04-Specs/phase-1-implementation-stack]].

## Sequence

```txt
Day-0 Onboarding Spec — done
→ Reconcile UX Spec — done
→ Phase 1 Technical Foundation ADR — this file
→ Phase 1 Implementation Stack — done
→ Phase 1 Data Model Spec
→ Implementation Plan
```

---

# Decisions

## Frontend baseline

```txt
React + TypeScript + Vite
```

Rationale:

- Phase 1 is an application, not an SEO/content product.
- SSR and Next.js server/client boundaries do not add meaningful value to Reconcile validation.
- React + Vite keeps local development and deployment simpler.

## Backend baseline

```txt
Java + Spring Boot
```

Rationale:

- the backend developer is more productive with Java and Spring
- Spring Boot provides REST, validation, transactions, persistence, security, and a future AI integration path
- using the team's strongest backend stack reduces risk under local network/package constraints

## Database

```txt
PostgreSQL
```

Use one PostgreSQL instance for application data and behavioral events.

Phase 1 does not require a separate analytics service or event platform.

## Authentication

Phase 1 uses one authentication identity:

```txt
Iranian phone number
```

Supported account-access paths:

```txt
phone-number OTP signup/login
JWT-based application sessions
password recovery/reset through verified phone OTP
```

Explicit exclusions:

```txt
Google OAuth
social login
email login
password login
multi-provider identity linking
```

Reason:

Google OAuth may be unreliable for the Iran-focused product because of sanctions and access restrictions. Phase 1 therefore keeps authentication dependent only on an Iranian SMS provider and the application's own JWT session system.

### Phone OTP flow

```txt
Enter phone number
→ Receive SMS OTP
→ Verify OTP
→ Create or authenticate user
→ Issue JWT-based session tokens
```

Auth happens before protected user data is created or loaded.

Day-0 onboarding starts immediately after successful authentication for a new user.

### OTP controls to define before implementation

Mahdi will define:

```txt
OTP expiration
resend cooldown
maximum verification attempts
request rate limit
single-use OTP invalidation
```

These are mandatory implementation decisions but are intentionally not assigned values in this ADR.

The exact SMS provider, JWT lifetimes, refresh-token storage, cookie settings, CORS rules, and CSRF strategy belong in the auth implementation plan.

## User timezone and day boundaries

Phase 1 is initially designed for users in Iran.

Default IANA timezone:

```txt
Asia/Tehran
```

Rules:

- store timestamps in UTC
- keep timezone as an explicit user-level value
- default new Phase 1 users to `Asia/Tehran`
- `plannedForDate` is a date-only value
- Reconcile compares against the user's local date, not server UTC date
- a task becomes eligible for Reconcile only after a new local calendar day starts
- never hardcode `UTC+03:30`

Detailed timezone fields and SQL/Java date types belong to the Data Model spec.

## Event log placement and ownership

Behavioral events use:

```txt
same PostgreSQL database
separate append-only events table
```

The detailed schema, metadata, enums, and indexes are owned by:

```txt
04-Specs/data-model-phase-1.md
```

## Atomic mutation principle

For state-changing actions, the domain mutation and corresponding event append must succeed atomically in the same transaction.

Examples:

```txt
carry task + append task_carried
complete task + append task_completed
drop task + append task_dropped
skip reconcile + append reconcile_skipped
```

The system must not persist one side without the other.

## API boundary

Frontend and backend communicate through a versioned REST API.

```txt
React client
→ Spring Boot REST API
→ PostgreSQL
```

Prefer same-site production routing:

```txt
https://app-domain/
https://app-domain/api/
```

This reduces cookie and CORS complexity for JWT sessions stored in HttpOnly cookies.

Phase 1 does not require GraphQL, WebSockets, microservices, or a separate analytics ingestion API.

## Deployment baseline

```txt
Docker-based deployment on a VPS
```

Expected production shape:

```txt
Nginx
frontend static build/container
Spring Boot backend container
PostgreSQL container or managed PostgreSQL
```

## PWA decision

```txt
Phase 1 PWA: Yes
Offline-first: No, deferred
```

Phase 1 may be installable as a basic PWA.

This decision does not include offline mutations, background sync, conflict resolution, or offline-first storage.

## AI boundary

Phase 1 is AI-ready at the schema and architecture level but does not implement runtime LLM functionality.

Keep:

```txt
source field
ai_generated enum value
AI guardrail/docs references
Phase 2 AI planning in the product map
```

Defer:

```txt
LLM provider integration
prompt execution
AI planning endpoints
structured-output runtime
AI feature flags
AI API calls
AI Knowledge Meter UI
```

---

# Explicit Exclusions

This ADR does not define:

- detailed User / Goal / Task / Event schema
- event metadata contracts
- exact OTP security values
- JWT lifetimes or refresh-token design
- SMS provider selection
- cookie/CORS/CSRF values
- offline-first synchronization
- microservices
- message queues
- advanced observability
- scaling architecture
- runtime AI infrastructure
- framework comparison essays

---

# Consequences

## Positive

- frontend and backend share one explicit baseline
- authentication has one identity anchor: normalized phone number
- no multi-provider identity table or account-linking logic is needed
- auth timing relative to Day-0 is clear
- Iran-local day boundaries are protected from server-UTC errors
- event-log ownership and transaction guarantees are clear
- future AI compatibility is preserved without building AI early

## Tradeoffs

- SMS delivery depends on a regional provider
- JWT lifecycle and revocation require a small security design
- cookie-based auth requires explicit Nginx, CORS, SameSite, Secure, and CSRF configuration
- timezone-aware day logic must be tested consistently
- VPS/Docker requires operational ownership
- offline-first value is postponed

These tradeoffs are accepted for Phase 1.

---

# Implementation Stack Reference

Frameworks, packages, testing tools, and scaffold rules are defined in:

```txt
04-Specs/phase-1-implementation-stack.md
```

# Next Step

Create and review:

```txt
04-Specs/data-model-phase-1.md
```

The Data Model spec should use:

```txt
User normalizedPhone as the single identity anchor
User timezone field
plannedForDate date-only type
Goal / Task / Event schema
detailed event metadata
transaction and indexing details
```