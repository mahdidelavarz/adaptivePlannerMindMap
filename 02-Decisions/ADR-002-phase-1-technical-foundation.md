# ADR-002 — Phase 1 Technical Foundation

## Status

Accepted for Phase 1.

## Purpose

Record the minimum shared technical assumptions needed before finalizing the Phase 1 Data Model and implementation plan.

This ADR is intentionally bounded. It documents the current build surface; it is not a framework comparison exercise or a long-term infrastructure blueprint.

## Sequence

```txt
Day-0 Onboarding Spec — done
→ Reconcile UX Spec — done
→ Phase 1 Technical Foundation ADR — this file
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
- SSR and Next.js server/client boundaries do not add meaningful value to the Reconcile validation.
- React + Vite keeps local development, deployment, and frontend ownership simpler.

This ADR records the chosen baseline. It does not reopen a React-vs-Next framework debate.

## Backend baseline

```txt
Java + Spring Boot
```

Rationale:

- The backend developer is more productive with Java and Spring.
- Spring Boot provides a stable base for REST APIs, validation, transactions, persistence, security, and later AI integration.
- Choosing the team's strongest backend stack reduces implementation risk under local package/network constraints.

## Database

```txt
PostgreSQL
```

Use one PostgreSQL instance for Phase 1 application data and behavioral events.

No separate analytics database or event platform is required for the closed 10–20 tester validation.

## Authentication

Phase 1 supports these authentication and account-access paths:

```txt
Phone-number OTP authentication
Google OAuth
JWT-based application sessions
Password recovery/reset through phone OTP only
```

### Phone OTP flow

```txt
Enter phone number
→ Receive OTP
→ Verify OTP
→ Create or authenticate user
→ Issue JWT-based session tokens
```

### Google OAuth flow

```txt
Continue with Google
→ Complete Google OAuth
→ Create or authenticate user
→ Issue JWT-based application session tokens
```

### Password recovery

Password recovery/reset must use verified phone OTP.

Do not introduce email-based recovery as an alternative Phase 1 recovery path.

Auth happens before protected user data is created or loaded.

Day-0 onboarding starts immediately after successful authentication for a new user, regardless of whether the user entered through phone OTP or Google OAuth.

### OTP controls to define before implementation

Mahdi will define these during the implementation/security step:

```txt
OTP expiration
Resend cooldown
Maximum verification attempts
Request rate limit
Single-use OTP invalidation
```

These items are mandatory implementation decisions but are intentionally not specified in this ADR.

The exact OTP/SMS provider, JWT lifetimes, refresh-token storage, and Google OAuth provider configuration belong in the implementation/security plan.

## User timezone and day boundaries

Phase 1 is initially designed for users in Iran.

Use the IANA timezone identifier:

```txt
Asia/Tehran
```

Do not hardcode a fixed UTC offset such as `UTC+03:30`.

Rules:

- Store timestamps in UTC.
- Interpret user-facing calendar days using the user's timezone.
- For the initial Iran-focused Phase 1 test, default the user's timezone to `Asia/Tehran`.
- Keep the timezone as an explicit user-level value so the system can support other regions later.
- `plannedForDate` is a date-only value with no time component.
- Reconcile day-boundary checks compare `plannedForDate` with the user's current local date, not the server's UTC date.
- A task becomes eligible for Reconcile only after a new local calendar day has started for that user.

Example:

```txt
plannedForDate = 2026-07-12
userTimeZone = Asia/Tehran
```

The backend must not treat that task as missed merely because the UTC date has already changed.

Detailed timezone fields and date types are owned by the Phase 1 Data Model spec.

## Event log placement and ownership

Behavioral events use:

```txt
same PostgreSQL database
separate append-only events table
```

The detailed event schema, required metadata, enums, indexes, and timezone fields are owned by:

```txt
04-Specs/data-model-phase-1.md
```

This ADR records only the architectural boundary so the same reasoning is not duplicated across files.

## Atomic mutation principle

For state-changing product actions, the domain mutation and its corresponding event append must succeed atomically in the same database transaction.

Examples:

```txt
carry task + append task_carried
complete task + append task_completed
drop task + append task_dropped
skip reconcile + append reconcile_skipped
```

The system must not leave task state updated without its event, or an event recorded without the matching state change.

## API boundary

Frontend and backend communicate through a versioned REST API.

```txt
React client
→ Spring Boot REST API
→ PostgreSQL
```

Phase 1 does not require GraphQL, WebSockets, microservices, or a separate analytics ingestion API.

## Deployment baseline

```txt
Docker-based deployment on a VPS
```

Expected production shape:

```txt
Nginx
Frontend container/static build
Spring Boot backend container
PostgreSQL container or managed PostgreSQL
```

The exact hosting provider and CI/CD workflow may remain implementation choices, provided the baseline stays simple and reproducible.

## PWA decision

```txt
Phase 1 PWA: Yes
Offline-first: No, deferred
```

Phase 1 may be installable as a basic PWA.

This ADR does not define caching strategy, offline mutations, background sync, conflict resolution, or offline fallback behavior.

## AI boundary

Phase 1 is AI-ready at the schema and architecture level, but does not implement runtime LLM functionality.

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
AI Knowledge meter UI
```

---

# Explicit exclusions

This ADR does not define:

- detailed User / Goal / Task / Event schema
- event metadata contracts
- exact OTP control values
- JWT token lifetimes or refresh-token design
- SMS provider selection
- Google OAuth provider configuration
- offline-first synchronization
- caching architecture
- microservices
- message queues
- advanced observability
- scaling architecture
- multi-region deployment
- production analytics pipeline
- runtime AI infrastructure
- framework comparison essays

Those are either owned by later specs or not justified by Phase 1.

---

# Consequences

## Positive

- frontend and backend share one explicit implementation baseline
- auth timing relative to Day-0 is clear
- phone OTP, Google OAuth, and OTP-based recovery paths are explicit
- Iran-local day boundaries are protected from server-UTC errors
- event-log ownership and transaction guarantees are clear
- Data Model can be finalized without hidden infrastructure assumptions
- future AI compatibility is preserved without building AI early

## Tradeoffs

- phone OTP delivery introduces a dependency on an SMS provider
- Google OAuth adds a second authentication integration path
- JWT lifecycle and revocation still require a small security design
- timezone-aware day logic must be implemented consistently across backend queries and tests
- VPS/Docker requires operational ownership
- offline-first value is postponed

These tradeoffs are accepted for Phase 1.

---

# Next step

Create and review:

```txt
04-Specs/data-model-phase-1.md
```

The Data Model spec should use this ADR as its technical baseline and own:

```txt
User timezone field
Phone / Google identity fields
plannedForDate date-only type
Goal / Task / Event schema
Detailed event metadata
Transaction and indexing details
```
