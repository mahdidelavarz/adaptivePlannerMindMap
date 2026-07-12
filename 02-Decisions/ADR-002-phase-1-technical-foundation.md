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

```txt
OTP-only authentication + JWT
```

Phase 1 auth rules:

- no password login
- no Google/social OAuth
- no multiple auth methods
- user enters the supported identifier and receives an OTP
- successful OTP verification creates/authenticates the user
- the backend issues JWT-based session tokens
- signup and login are the same OTP flow from the user's perspective

Auth happens before protected user data is created or loaded.

Day-0 onboarding starts immediately after successful authentication for a new user.

The exact OTP provider, token lifetimes, refresh-token storage, retry limits, and abuse controls belong in the implementation/security plan, not this ADR.

## Event log placement and ownership

Behavioral events use:

```txt
same PostgreSQL database
separate append-only events table
```

The detailed event schema, required metadata, enums, and indexes are owned by:

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

- detailed Goal / Task / Event schema
- event metadata contracts
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
- event-log ownership and transaction guarantees are clear
- Data Model can be finalized without hidden infrastructure assumptions
- future AI compatibility is preserved without building AI early

## Tradeoffs

- OTP delivery introduces a dependency on an external provider
- JWT lifecycle and revocation still require a small security design
- VPS/Docker requires operational ownership
- offline-first value is postponed

These tradeoffs are accepted for Phase 1.

---

# Next step

Create and review:

```txt
04-Specs/data-model-phase-1.md
```

The Data Model spec should use this ADR as its technical baseline and own the detailed event schema.