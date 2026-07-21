# Phase 1 Implementation Plan

## Status

Legacy Phase 1 implementation plan — non-authoritative for the current AI-native MVP.

Only the development and pilot-operation rules explicitly retained in [[01-Closed-Discussions/001-008-legacy-surviving-decisions]] survive. Current baseline reconciliation, reuse classification, milestones, and rollout gates are owned by [[01-Open-Discussions/022-updated-mvp-implementation-plan]].

## Purpose

Define the build sequence, team ownership, integration gates, test gates, deployment readiness, and pilot release criteria for the accepted Phase 1 scope.

Use with:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/auth-phase-1]]
- [[04-Specs/api-contracts-phase-1]]
- [[04-Specs/phase-1-implementation-stack]]
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]

---

# 1. Team Model

This plan assumes exactly three people:

```txt
Mahdi
- frontend implementation
- frontend architecture
- product/spec ownership
- API client and state integration
- E2E coordination

Backend developer
- Java/Spring/PostgreSQL
- migrations and persistence
- transactions and event atomicity
- security and authentication
- backend deployment

Designer
- UX/UI
- interaction states
- responsive behavior
- usability review
- design handoff
```

The designer is not counted as a third implementation stream.

The project therefore has:

```txt
2 engineering streams
1 design stream
```

The design stream stays slightly ahead of implementation and works from formal UX and API states.

---

# 2. Execution Principles

- build bounded vertical slices rather than finishing all backend before all frontend
- use accepted Markdown specs as the design authority
- generate OpenAPI from Spring controllers as living implementation documentation
- do not maintain a second independent hand-written OpenAPI source
- prioritize transaction, concurrency, timezone, auth, and event-integrity tests
- do not add infrastructure without current Phase 1 evidence
- every slice must include backend rules, frontend states, tests, and accepted event behavior

Excluded infrastructure:

```txt
Redis
message queues
microservices
Kubernetes
GraphQL
WebSockets
offline mutation sync
AI runtime
refresh-token families
analytics warehouse
```

---

# 3. Actual Critical Path

The plan is not strictly sequential after Foundation.

```txt
Foundation + Database
→ dev test session available
→ Auth and early product slices partially parallel
→ User/Onboarding
→ Goal/Task Core
→ Today
→ Reconcile G1 backend correctness
→ Reconcile G2 frontend/E2E
→ APP_OPENED shipped behavior
→ Production infrastructure + PWA
→ validation scripts + release gate
```

---

# 4. Development-Only Test Session

Implement:

```txt
POST /api/v1/dev/test-session
```

Purpose:

- unblock frontend and domain integration before real OTP work is complete
- mint the same JWT cookie used by production auth
- create or load a fixed development test user

Rules:

- enabled only in `local` and `test` profiles
- unavailable in `production`
- production profile must return 404
- must not appear as a product endpoint
- must have an integration test proving it cannot run in production

---

# 5. Phase A — Repository and Engineering Foundation

## Backend

- Java 21 / Spring Boot 4.1 project
- Maven Wrapper
- accepted dependencies only
- package-by-feature baseline
- local/test/production profiles
- PostgreSQL and Flyway configuration
- Hibernate schema validation outside tests
- Actuator health/info
- generated OpenAPI in local/test
- global Problem Details handling
- trace/correlation ID
- deterministic Clock/timezone test support

## Frontend

- React 19 + TypeScript + Vite
- strict TypeScript settings
- React Router
- TanStack Query
- Axios `/api/v1` client with credentials
- Zustand conventions
- React Hook Form + Zod
- Tailwind and minimal UI primitives
- route/error/loading conventions
- Vitest, Testing Library, MSW, Playwright
- ESLint and Prettier

## Infrastructure

- local Docker Compose for PostgreSQL
- environment templates
- initial same-origin Nginx proxy skeleton
- GitHub Actions for frontend and backend checks
- pinned versions

## Exit gate

```txt
frontend starts
backend starts
Flyway applies
frontend reaches backend through accepted /api shape
CI runs lint, typecheck, tests, build, Maven verify
Problem Details has an integration test
```

---

# 6. Phase B — Database Foundation

Use migrations:

```txt
V1: users, otp_challenges, behavioral_events, identity/event indexes and constraints
V2: goals, tasks, domain indexes and constraints
```

Required:

- consistent UUID strategy
- UTC timestamps
- `users.normalized_phone` unique constraint
- `users.session_epoch >= 0`
- Task status/timestamp/carry checks
- accepted indexes
- append-only BehavioralEvent persistence convention
- manual mapping conventions

## Exit gate

- migrations apply on empty PostgreSQL
- Hibernate validates schema
- unique phone constraint is proven
- contradictory Task states are rejected
- event persistence does not expose ordinary update/delete operations

Milestone 1 includes Phases A and B together.

---

# 7. Phase C — Authentication

## Backend

- Iranian phone normalizer
- OTP challenge persistence
- HMAC-SHA-256 digest
- `OtpDeliveryGateway`
- fake/local SMS adapter first
- provider adapter boundary
- OTP request and cooldown contract
- OTP verification transaction
- OTP row locking and single use
- duplicate-user race recovery
- single JWT issuance after commit
- `sessionEpoch` validation
- HttpOnly cookie policy
- Origin/Referer checks
- logout
- `/users/me`

Forced revocation is operational, not a Phase 1 UI feature.

Use a small internal script or backend command that increments `sessionEpoch`. Avoid undocumented routine SQL edits.

## Frontend

- phone form
- OTP input
- cooldown from `resendAvailableAt`
- auth errors
- `/users/me` bootstrap
- onboarding-status routing
- logout
- expired/revoked session behavior

## Exit gate

- auth race tests pass
- cookie/security tests pass
- request OTP → verify → reload → logout works E2E
- no refresh-token/password/OAuth machinery exists

Auth and early product work may proceed partially in parallel after dev test-session exists.

---

# 8. Phase D — User and Day-0 Onboarding

## Backend

- `PATCH /users/me`
- display-name and IANA timezone validation
- onboarding start command + event atomically
- onboarding complete command + event atomically
- idempotent commands

## Frontend

- Day-0 flow
- optional profile fields
- optional Goal
- optional Task
- optional Today scheduling
- empty completion path

## Exit gate

- onboarding works with no Goal and no Task
- reload restores progress
- repeated start/complete creates no duplicate events
- completed user does not re-enter Day-0

---

# 9. Phase E — Goal and Task Core

## In pilot scope

Goal:

- create
- list
- get
- update title
- use in Task forms

Task:

- create
- list/filter
- get
- update ACTIVE Task
- same-user Goal ownership
- complete outside Reconcile
- accepted event mapping
- guarded mutations

## Deferred until after pilot

```txt
Goal deletion
Goal deletion UI
linked-Task detachment behavior
```

The API spec may retain the future contract, but deletion is not part of the validation candidate.

## Task creation event rule

When creating a Task already containing `plannedForDate`:

```txt
append TASK_CREATED only
include initial plannedForDate in TASK_CREATED metadata
```

Do not append `TASK_PLANNED` for the same creation command.

## Exit gate

- Task and event mutation is atomic
- title-only edit creates no event
- planned-date mutation creates exactly the accepted event
- terminal Task cannot edit/reopen
- duplicate complete cannot duplicate events
- cross-user Goal references fail safely

---

# 10. Phase F — Today

## Backend

- `GET /today`
- use User timezone and local date
- return ACTIVE Tasks planned exactly for local today
- valid empty response

## Frontend

- Today route
- loading/error/empty states
- create Task
- complete Task
- edit navigation
- non-shaming language

## Exit gate

- Asia/Tehran boundary tests pass
- server UTC date is not used as user day
- unscheduled, past, future, completed, and dropped Tasks are excluded

---

# 11. Phase G1 — Reconcile Backend Correctness

This is the highest-risk backend slice and has its own exit gate.

## Work

- eligibility query
- `POST /reconcile/open`
- once-per-day `RECONCILE_SHOWN`
- no-same-day-retrigger lookup
- `POST /reconcile/skip`
- idempotent `RECONCILE_SKIPPED`
- unified resolve command
- Done, Carry, Drop
- first successful resolve appends `RECONCILE_STARTED` once
- fresh `remainingCount`
- fresh eligibility query before completion
- one `RECONCILE_COMPLETED` only when current DB state is empty
- return remaining straggler Tasks/count when present

## Mandatory tests before frontend integration

```txt
eligibility uses user local date
shown recorded once per day
skip blocks same-day retrigger
first resolve records started once
Done/Carry/Drop mutation and event are atomic
fresh straggler prevents premature completion
completion event emitted once
concurrent resolve cannot duplicate terminal events
rollback preserves domain/event consistency
```

## Exit gate

All G1 integration and concurrency tests must be green before G2 is considered integrated.

---

# 12. Phase G2 — Reconcile Frontend and E2E

## Work

- open Reconcile via command endpoint
- accepted available/unavailable states
- Done
- Carry with Today default
- frontend-delayed Drop
- 4–6 second Undo window
- cancel pending DROP when Undo is pressed
- no `sendBeacon` flush on tab close
- Not now / skip
- completion driven only by backend response
- never infer terminal completion from local array length

## E2E gate

```txt
Done
Carry
Drop + Undo sends no request
Drop after timer persists
skip
no same-day retrigger
next-local-day retrigger
backend-returned straggler continues flow
```

---

# 13. Phase H1 — App Presence

Implement:

```txt
POST /app/open
APP_OPENED at most once per user local day
```

This measures daily presence, not session count.

Required:

- event correctness
- local-day idempotency
- no bundled bootstrap response

This shipped code belongs before the pilot deployment.

---

# 14. Phase H2 — Validation SQL and Scripts

Use direct SQL or small scripts, not an admin dashboard.

Required metrics:

- D1 / D3 / D7 return
- return after absence/slippage
- Reconcile shown/started/skipped/completed
- Done/Carry/Drop distribution
- carry-count patterns
- meaningful action within five minutes after skip

H2 does not block infrastructure deployment, but core queries must be implemented and verified against seeded data before pilot start.

---

# 15. Phase I — Production Infrastructure and PWA

## Infrastructure

- backend production Dockerfile
- frontend production build/serving strategy
- PostgreSQL production configuration
- Nginx
- TLS
- `/api` proxy
- SPA fallback
- compression and security headers
- production env templates
- logging
- Actuator health
- backup and restore procedure
- deployment runbook

## External monitoring

Add an external uptime check against the deployed health endpoint.

Requirements:

- checks from outside the VPS
- alerts reach a channel the team actively watches
- monitoring is tested before pilot start

## PWA

Keep late:

- manifest
- icons
- static-asset caching
- update notification

Still excluded:

- offline mutations
- background sync
- conflict resolution

## Exit gate

- cookies and Origin/Referer behavior work behind Nginx
- SPA routes refresh
- `/api` proxy preserves headers
- Flyway runs safely
- health checks pass
- external monitoring alerts correctly
- PWA installs
- no offline mutation behavior exists

---

# 16. Phase J — Validation Readiness

Before pilot:

- define OTP expiry, cooldown, attempts, rate limits, code length
- define JWT lifetime and cookie name
- confirm SMS provider cost and behavior
- define provider timeout/retry
- create tester onboarding instructions
- define support and bug severity process
- prepare feedback/interview script
- define start/end dates
- freeze major upgrades during validation
- rehearse backup restore
- complete production smoke test

## Blocking release gate

```txt
critical E2E flows pass
atomic mutation/event tests pass
auth race tests pass
timezone boundary tests pass
Reconcile fresh-completion race test passes
production cookie/CSRF checks pass
metrics queries pass against seeded data
event payload privacy review passes
OTP resend/rate-limit/max-attempt behavior is verified against production config
backup exists and restore is rehearsed
external uptime monitoring is active and alerting
```

The pilot does not start before every blocking item passes.

---

# 17. Team Parallelization

## Backend developer

Early:

- Phase A backend
- Phase B migrations
- dev test-session
- Phase C auth backend

Then:

- onboarding APIs
- Goal/Task APIs
- Today
- Reconcile G1
- deployment backend

## Mahdi

Early:

- Phase A frontend
- API client/error handling
- contract-accurate MSW fixtures
- auth UI
- onboarding UI using dev test-session

Then:

- Goal/Task
- Today
- Reconcile G2
- Playwright and integration coordination

## Designer

Works slightly ahead on:

- auth screens and OTP error states
- Day-0
- Today
- Goal/Task forms
- Reconcile available/unavailable/resolution states
- loading, empty, error, and success states
- responsive behavior

The designer must use formal API and UX contracts and must not invent new fields, routes, or product behavior.

---

# 18. Milestones

## Milestone 1 — Running Skeleton

```txt
Phase A + Phase B
```

Demo: frontend → backend → PostgreSQL.

## Milestone 2 — Parallel Development Enabled

```txt
dev test-session + Auth foundation
```

Demo: frontend/domain work can integrate without waiting for real SMS completion.

## Milestone 3 — First Useful Planner

```txt
User/Onboarding + Goal/Task + Today
```

Demo: onboard → create Goal/Task → plan → complete.

## Milestone 4 — MVP Core Loop

```txt
Reconcile G1 + G2
```

Demo: missed Task → Done/Carry/Drop/Skip with correct events.

## Milestone 5 — Validation Candidate

```txt
H1 + H2 + I + J
```

Demo: deployed, monitored, measurable 14-day pilot candidate.

---

# 19. Definition of Done Per Slice

A slice is done only when:

```txt
accepted API implemented
backend validation and ownership enforced
state + event transaction covered
frontend loading/empty/error/success states covered
contract/integration tests present
critical E2E updated
OpenAPI reflects controllers
no out-of-scope feature introduced
```

---

# 20. Test Timing

Mandatory before frontend integration:

```txt
migrations and constraints
atomic rollback tests
unique-phone/user race tests
timezone boundary tests
Reconcile fresh-eligibility test
state/event consistency
```

Mandatory before validation, but not required to begin frontend integration:

```txt
exact production OTP limits
production cookie attributes
PWA install/update
external uptime alerts
production deployment smoke tests
privacy review
backup restore rehearsal
```

---

# 21. Post-Pilot Work

Explicitly deferred:

```txt
Goal deletion
admin dashboard
per-device sessions
refresh tokens
offline-first mutations
AI runtime
Weekly Review
calendar integration
```

These require new evidence or a new accepted discussion.
