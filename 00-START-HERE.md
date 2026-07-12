# Start Here — Phase 1 Implementation

## Current State

```txt
Phase 0 — Definition & Architecture: Closed
Phase 1 — Implementation: Ready to Start
```

Read [[02-Decisions/ADR-003-phase-0-closure]] before proposing changes to accepted scope or architecture.

## Required Reading Order

### Everyone

1. [[02-Decisions/ADR-003-phase-0-closure]]
2. [[05-Implementation/phase-1-plan]]
3. [[04-Specs/api-contracts-phase-1]]

### Mahdi

4. [[04-Specs/day-0-onboarding]]
5. [[04-Specs/reconcile-ux]]
6. [[04-Specs/phase-1-implementation-stack]]
7. [[04-Specs/data-model-phase-1]]
8. [[04-Specs/auth-phase-1]]

### Backend Developer

4. [[02-Decisions/ADR-002-phase-1-technical-foundation]]
5. [[04-Specs/phase-1-implementation-stack]]
6. [[04-Specs/data-model-phase-1]]
7. [[04-Specs/auth-phase-1]]
8. [[04-Specs/reconcile-ux]]

### Designer

4. [[04-Specs/day-0-onboarding]]
5. [[04-Specs/reconcile-ux]]
6. [[04-Specs/api-contracts-phase-1]]

The designer should focus on real states from the contracts:

```txt
loading
empty
validation error
auth error
session expired/revoked
Reconcile available/unavailable
Done / Carry / Drop / Skip
remainingCount
responsive behavior
```

## First Active Milestone

```txt
Milestone 1 — Running Skeleton
```

From [[05-Implementation/phase-1-plan]]:

```txt
Phase A — Repository and Engineering Foundation
Phase B — Database Foundation
```

### Mahdi starts with

- React + Vite + strict TypeScript scaffold
- router and route shell
- TanStack Query
- Axios `/api/v1` client with credentials
- RHF + Zod
- Tailwind baseline
- Vitest, Testing Library, MSW, Playwright
- lint and formatting

### Backend developer starts with

- Java 21 / Spring Boot 4.1 scaffold
- Maven Wrapper
- package-by-feature structure
- local/test/production profiles
- PostgreSQL + Flyway
- Problem Details
- trace ID
- Actuator
- Testcontainers
- V1 and V2 schema migrations

### Designer starts with

- visual foundation and reusable primitives
- Auth screens and OTP states
- Day-0 flow
- Today states
- Goal/Task forms
- Reconcile states
- empty/loading/error/responsive variants

## First Milestone Exit Gate

Milestone 1 is complete only when:

```txt
frontend starts
backend starts
PostgreSQL starts
Flyway migrations apply
frontend calls backend through the accepted /api shape
CI is green for both projects
Problem Details has an integration test
```

## Change Rule During Coding

Do not silently change accepted behavior because implementation becomes inconvenient.

Use this test:

```txt
Does the change preserve the accepted API, model, event, auth, UX, and Phase 1 scope?
```

- Yes: treat it as an implementation detail.
- No: open a bounded discussion before changing code and formal specs.

## Superseded Historical Ideas

Do not implement from older notes that mention:

```txt
password recovery
refresh tokens or rotating refresh sessions
Google OAuth or social login
multiple user identities
Redis
AI runtime
Weekly Review
Goal deletion before pilot
admin analytics dashboard
offline-first mutation sync
```

The accepted formal specs and implementation plan override those historical proposals.

## Next Build Step

Create or prepare the actual implementation repository structure:

```txt
adaptive-planner/
  frontend/
  backend/
  infra/
    nginx/
    docker/
  docs/
```

Then execute Milestone 1 without reopening Phase 0.