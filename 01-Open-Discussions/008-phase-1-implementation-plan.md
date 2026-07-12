# Discussion 008 — Phase 1 Implementation Plan

## Status

Waiting for Claude review.

## Context

The Phase 1 product and technical foundation are now formalized through:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/auth-phase-1]]
- [[04-Specs/api-contracts-phase-1]]
- [[04-Specs/phase-1-implementation-stack]]
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]

The remaining planning problem is not what to build. It is how to sequence implementation so frontend, backend, database, infrastructure, testing, and validation preparation converge without avoidable rework.

This document proposes a dependency-aware implementation plan for review.

It does not reopen product scope, API contracts, authentication architecture, data-model decisions, or the technology stack unless the proposed sequence exposes a direct contradiction.

---

# Planning Principles

## 1. Build vertical slices, not isolated technical layers

Avoid this sequence:

```txt
finish all backend
→ finish all frontend
→ integrate at the end
```

Prefer bounded vertical slices where the API, persistence, frontend state, UI, tests, and event behavior become usable together.

Reason:

- contract mistakes surface earlier
- frontend and backend developers can validate assumptions continuously
- each milestone produces something demonstrable
- integration does not become a final-week archaeological excavation

## 2. Infrastructure before product complexity

Local startup, migrations, configuration, API error format, authentication boundary, and CI must become reliable before feature work expands.

## 3. Accepted specs are the source of truth

Implementation must not silently invent:

- new fields
- new endpoints
- new events
- new auth mechanisms
- new Phase 1 features

Any contradiction found during coding returns to a bounded discussion or spec correction.

## 4. Tests follow risk, not file count

Prioritize tests for:

- atomic state + event mutations
- OTP concurrency
- user-creation race handling
- timezone/local-date behavior
- Reconcile no-retrigger
- fresh eligibility before completion
- auth-cookie and CSRF boundaries

Do not chase superficial percentage coverage while transaction boundaries remain untested.

## 5. No future infrastructure without current evidence

Do not add:

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

# Proposed Delivery Shape

One repository:

```txt
adaptive-planner/
  frontend/
  backend/
  infra/
    nginx/
    docker/
  docs/
```

Primary environments:

```txt
local
test
production
```

Primary execution order:

```txt
Foundation
→ Auth
→ User + Onboarding
→ Goal + Task Core
→ Today
→ Reconcile
→ App Presence + Measurement
→ PWA + Production Infrastructure
→ Validation Readiness
```

---

# Phase A — Repository and Engineering Foundation

## Goal

Create a reproducible project that starts locally, runs checks in CI, and exposes one functioning frontend-to-backend request path.

## Backend work

- create Java 21 / Spring Boot 4.1 Maven project
- add accepted Spring dependencies only
- configure package-by-feature baseline
- add `local`, `test`, and `production` profiles
- configure PostgreSQL connection
- configure Flyway
- set Hibernate production behavior to schema validation
- add Actuator health/info
- add springdoc OpenAPI in local/test
- implement global Problem Details exception handling
- add request trace/correlation ID
- define shared clock/timezone abstractions where tests require deterministic time

## Frontend work

- create React 19 + TypeScript + Vite project
- enable accepted strict TypeScript flags
- configure React Router
- configure TanStack Query
- configure Axios `/api/v1` client with credentials
- configure Zustand conventions
- configure React Hook Form + Zod
- configure Tailwind and minimal UI primitives
- configure error-boundary and route-loading conventions
- configure Vitest, Testing Library, MSW, and Playwright
- configure ESLint and Prettier

## Infrastructure work

- create local Docker Compose for PostgreSQL
- add backend and frontend development environment templates
- add initial Nginx same-origin proxy skeleton
- add GitHub Actions checks for frontend and backend
- pin major/exact versions according to stack policy

## Required proof

```txt
frontend loads
backend starts
Flyway applies an initial migration
frontend calls /api/v1/health or equivalent proxied endpoint
CI runs lint, typecheck, tests, build, and Maven verify
```

## Exit criteria

- a new developer can clone, configure, and start the system from documented steps
- no secrets are committed
- local frontend/backend communication matches the future `/api` proxy shape
- Problem Details format is covered by at least one integration test

---

# Phase B — Database Foundation and Shared Persistence Rules

## Goal

Create the accepted schema and common persistence behavior before feature services depend on it.

## Work

- create Flyway migrations for:
  - `users`
  - `otp_challenges`
  - `goals`
  - `tasks`
  - `behavioral_events`
- add UUID strategy consistently
- add UTC timestamp columns
- add accepted indexes
- add `users.normalized_phone` unique constraint
- add `users.session_epoch` with non-negative check
- add Task status/timestamp/carry-count checks
- add append-only event persistence convention
- add repositories or persistence adapters by feature
- define manual DTO/domain/entity mapping conventions

## Important implementation question

Should all tables be created in one initial migration or split by feature migrations?

GPT preference:

```txt
V1 core identity + event foundation
V2 goal/task domain
```

Reason:

- still simple
- keeps auth and product-domain changes understandable
- avoids an artificial migration per table

## Required tests

- migrations apply on empty PostgreSQL
- Hibernate validates the migrated schema
- unique phone constraint is real
- Task database checks reject contradictory state
- event rows can be inserted but are not exposed to ordinary update/delete repository operations

## Exit criteria

The schema supports Auth, User, Goal, Task, and BehavioralEvent implementation without another structural design decision.

---

# Phase C — Authentication Vertical Slice

## Goal

A user can request an OTP, verify it, receive a valid cookie session, reload, and log out.

## Backend work

- implement Iranian phone normalizer
- implement OTP challenge persistence
- implement HMAC-SHA-256 OTP digest
- implement `OtpDeliveryGateway`
- create fake/local SMS adapter
- create production provider adapter boundary without selecting accidental SDK coupling
- implement OTP request endpoint
- implement request-rate and resend-cooldown response contract
- implement OTP verification transaction
- lock/consume challenge atomically
- create/load User safely under normalized-phone race
- recover from unique-constraint race without HTTP 500
- issue single JWT after transaction commit
- validate `sessionEpoch` on authenticated requests
- configure HttpOnly cookie attributes
- implement Origin/Referer checks for unsafe requests
- implement logout
- implement logout-all / forced revoke endpoint
- implement `/users/me`

## Frontend work

- auth route and phone form
- OTP input flow
- resend cooldown from `resendAvailableAt`
- generic errors without account-existence leakage
- authenticated bootstrap through `/users/me`
- route based on `onboardingStatus`
- logout behavior
- revoked/expired-session handling

## Required backend tests

All tests listed in [[04-Specs/auth-phase-1]], especially:

```txt
phone normalization
OTP expiry and single use
concurrent OTP consumption
concurrent new-user verification
unique-constraint race recovery
JWT cookie attributes
sessionEpoch validation
cross-origin mutation rejection
```

## Required E2E proof

```txt
request OTP
verify as new user
enter onboarding
reload and remain authenticated
logout and return to auth
```

## Exit criteria

Auth works end to end with the fake SMS adapter and no refresh-token or password machinery exists.

---

# Phase D — User Profile and Day-0 Onboarding

## Goal

A new authenticated user can complete Day-0 with every creation step optional.

## Backend work

- implement `PATCH /users/me`
- validate display-name length and IANA timezone
- implement onboarding start command
- append `ONBOARDING_STARTED` atomically
- implement onboarding complete command
- append `ONBOARDING_COMPLETED` atomically
- keep both commands idempotent

## Frontend work

- onboarding shell based on accepted Day-0 spec
- optional display-name/timezone step if present in final UI
- optional Goal creation
- optional Task creation
- optional scheduling for Today
- empty completion path
- no forced Goal or Task creation

## Required tests

- user can complete onboarding with no Goal and no Task
- repeated start/complete does not duplicate events
- reload during onboarding restores correct state
- completed users do not return to Day-0

## Exit criteria

The full new-user route from OTP verification to Today works even when the user skips every optional creation step.

---

# Phase E — Goal and Task Core

## Goal

Implement the smallest reliable planning surface independent of Reconcile.

## Goal slice

Backend:

- create Goal
- list Goals
- get Goal
- update Goal title
- delete Goal transactionally
- set linked Task `goalId` to null on Goal deletion
- preserve behavioral events

Frontend:

- Goal list
- Goal create/edit
- Goal selection primitive used by Task forms
- Goal deletion UX with clear consequence for linked Tasks

## Task slice

Backend:

- create Task
- list/filter Tasks
- get Task
- update ACTIVE Task
- enforce same-user Goal ownership
- implement explicit `plannedForDate` event mapping
- complete ACTIVE Task outside Reconcile
- use guarded state mutations
- append accepted events atomically

Frontend:

- Task list
- standalone and Goal-linked Task creation
- optional scheduling
- Task edit
- complete action
- minimal nullable Goal summary rendering

## Important event question

When a Task is created already carrying `plannedForDate`, should implementation append only `TASK_CREATED` with planning metadata, or both `TASK_CREATED` and `TASK_PLANNED`?

This is not fully explicit in the current API spec.

GPT preference:

```txt
append TASK_CREATED only
include initial plannedForDate in TASK_CREATED metadata
```

Reason:

- one user command should not automatically produce two semantic events unless analysis specifically needs both
- later scheduling mutations still use `TASK_PLANNED`
- avoids inflated planning-event counts

Claude should review this before formalization.

## Required tests

- Goal ownership cannot be crossed between users
- Goal deletion preserves Tasks as standalone
- Task creation and event append are atomic
- Task PATCH mapping emits exactly the accepted event
- title-only Task edit emits no event
- terminal Task cannot be edited or reopened
- duplicate complete attempts cannot duplicate events

## Exit criteria

A returning user can maintain Goals and Tasks and safely plan work for local dates without Reconcile.

---

# Phase F — Today Vertical Slice

## Goal

Provide the normal daily planning view before adding missed-work reconciliation.

## Backend work

- implement `GET /today`
- calculate current local date using `User.timezone`
- return ACTIVE Tasks planned exactly for that local date
- return valid empty response

## Frontend work

- Today route
- loading/error/empty states
- create Task from Today
- complete Task from Today
- navigate to edit Task
- avoid guilt/failure language

## Required tests

- Asia/Tehran day-boundary behavior
- Today does not use server UTC date
- unscheduled, completed, dropped, future, and past Tasks are excluded
- empty Today remains usable

## Exit criteria

Today is functional as a standalone daily view and can be exercised before Reconcile exists.

---

# Phase G — Reconcile Vertical Slice

## Goal

Implement the central Phase 1 product loop with correct events, no same-day retrigger, and safe completion behavior.

## Backend work

- implement Reconcile eligibility query
- implement `POST /reconcile/open`
- append `RECONCILE_SHOWN` at most once per local day
- implement no-same-day-retrigger lookup
- implement `POST /reconcile/skip`
- append `RECONCILE_SKIPPED` idempotently
- implement unified resolve command
- implement Done
- implement Carry
- implement Drop
- append `RECONCILE_STARTED` on first successful resolution that day
- return fresh `remainingCount`
- before completion, rerun eligibility query inside the same transaction
- append `RECONCILE_COMPLETED` once only when fresh eligibility is empty
- return updated remaining tasks/count if a new eligible straggler exists

## Frontend work

- detect/open Reconcile through the command endpoint
- render unresolved tasks sequentially or in accepted UX shape
- implement Done
- implement Carry date selection with Today default
- implement frontend-delayed Drop
- show Undo toast for 4–6 seconds
- send DROP only after timer expires
- cancel pending request on Undo
- do not use `sendBeacon` on close
- implement Not now / skip
- complete transition only from backend response
- never derive completion from local array length alone

## Required tests

Backend:

```txt
eligibility uses user's local date
shown is recorded once per day
skip prevents same-day retrigger
first resolve records started once
Done/Carry/Drop mutation + event are atomic
fresh straggler prevents premature completion
completion event is emitted once
concurrent resolve cannot duplicate terminal events
```

Frontend/E2E:

```txt
Done flow
Carry flow
Drop + Undo sends no request
Drop after timer persists
skip flow
no same-day retrigger
next-local-day retrigger
new remaining task returned by backend continues the flow
```

## Exit criteria

The accepted Reconcile UX can be tested end to end against real PostgreSQL with deterministic timezone and event behavior.

---

# Phase H — App Presence and Validation Measurement

## Goal

Capture the minimum trustworthy data needed for the 14-day validation without building an analytics platform.

## Work

- implement `POST /app/open`
- append `APP_OPENED` once per user local day
- keep it separate from bootstrap data
- verify all accepted BehavioralEvent metadata contracts
- add SQL queries or small internal scripts for:
  - D1 / D3 / D7 return
  - Reconcile shown / started / skipped / completed
  - Done / Carry / Drop distribution
  - carry-count patterns
  - meaningful action within five minutes after skip
  - return after slippage
- define test-user exclusion/filter strategy if internal team accounts exist
- define timezone-safe aggregation rules

## Open question

Should Phase 1 include a small admin-only metrics endpoint/dashboard, or should analysis use direct SQL exports for 10–20 testers?

GPT preference:

```txt
direct SQL queries or scripts
no admin dashboard in the initial build
```

Reason:

- the validation group is tiny
- dashboard UI is not user value
- queries can be reviewed against the exact metric definitions

## Exit criteria

The team can answer the accepted validation questions from recorded data without manual reconstruction from production logs.

---

# Phase I — PWA and Production Infrastructure

## Goal

Deploy a reliable installable web app while keeping offline-first behavior excluded.

## Frontend work

- configure manifest
- add application icons
- configure basic static-asset caching
- add update notification behavior
- verify no API mutation is cached as an offline queue

## Backend and infrastructure work

- production Dockerfile for backend
- production frontend build/container or static serving strategy
- PostgreSQL service/managed configuration
- production Nginx config
- TLS
- `/api` proxy
- SPA fallback
- compression
- security headers
- environment-variable templates
- production logging
- Actuator health check
- backup/restore procedure for PostgreSQL
- deployment runbook

## Required deployment verification

```txt
cookie Secure/SameSite behavior works behind Nginx
Origin/Referer checks accept the real app origin
SPA routes refresh correctly
/api proxy preserves required headers
Flyway runs safely
health checks pass
PWA installs
no offline mutation behavior exists
```

## Exit criteria

A production-like environment supports the entire E2E flow through Nginx and HTTPS.

---

# Phase J — Validation Readiness and Release Gate

## Goal

Move from “features appear finished” to “the 14-day validation can start without knowingly corrupting its result.”

## Required work

- create test accounts or onboarding instructions
- confirm SMS provider behavior and cost
- define OTP values:
  - expiration
  - resend cooldown
  - maximum attempts
  - request rate limits
  - code length
- define JWT lifetime and cookie name
- define provider timeout/retry behavior
- create tester support process
- create issue severity categories
- define blocking vs non-blocking bugs
- verify privacy/consent copy for behavioral events
- verify event payloads contain no unnecessary private data
- prepare SQL/reporting queries
- prepare feedback/interview script
- define test start/end dates
- freeze major dependency upgrades during validation
- perform backup restore rehearsal
- complete release smoke test

## Release gate

The validation does not start until all are true:

```txt
critical E2E flows pass
atomic mutation/event tests pass
auth race tests pass
timezone boundary tests pass
production cookie/CSRF checks pass
Reconcile completion race test passes
metrics queries return expected seeded results
backup exists and restore procedure is known
```

## Exit criteria

The product is not merely deployable; it is trustworthy enough that tester behavior can be interpreted without wondering whether missing events, auth failures, or timezone bugs fabricated the outcome.

---

# Proposed Milestones

## Milestone 1 — Running Skeleton

Includes:

```txt
Phase A
Phase B
```

Demo:

```txt
frontend → backend → PostgreSQL through local environment
```

## Milestone 2 — Authenticated User

Includes:

```txt
Phase C
```

Demo:

```txt
OTP → cookie session → /users/me → logout
```

## Milestone 3 — First Useful Planner

Includes:

```txt
Phase D
Phase E
Phase F
```

Demo:

```txt
onboarding → create Goal/Task → Today → complete Task
```

## Milestone 4 — MVP Core Loop

Includes:

```txt
Phase G
```

Demo:

```txt
missed Task → Reconcile → Done/Carry/Drop/Skip → correct events
```

## Milestone 5 — Validation Candidate

Includes:

```txt
Phase H
Phase I
Phase J
```

Demo:

```txt
production deployment + measurable 14-day pilot readiness
```

---

# Parallelization Proposal

Assumed team:

```txt
Mahdi: frontend + product/spec ownership
Backend developer: Java/Spring/PostgreSQL
Designer: UX/UI and visual system
```

## Early parallel work

Backend developer:

- Phase A backend
- Phase B schema
- Phase C auth backend

Mahdi:

- Phase A frontend
- API client/error conventions
- auth and onboarding UI

Designer:

- auth screens
- Day-0 flow
- Today
- Goal/Task forms
- Reconcile states
- empty/loading/error states

## Integration rule

Each vertical slice should have a shared contract fixture or MSW mock before both sides finish.

Do not let frontend mocks invent shapes that differ from [[04-Specs/api-contracts-phase-1]].

## Suggested ownership boundary

```txt
Backend owns:
- migrations
- persistence
- transactions
- security
- API implementation
- backend integration tests

Mahdi owns:
- frontend architecture
- client API modules
- React Query/cache behavior
- forms and routing
- E2E coordination

Designer owns:
- interaction and visual design
- responsive behavior
- state coverage
- usability review

Shared:
- API contract changes
- acceptance criteria
- bug triage
- release gate
```

---

# Definition of Done Per Slice

A feature is not done because one layer compiles.

For every slice, require:

```txt
accepted API implemented
backend validation implemented
ownership/security enforced
state + event transaction covered
frontend loading/empty/error/success states covered
contract test or integration test present
critical E2E path updated
OpenAPI reflects the contract
no out-of-scope capability introduced
```

---

# Proposed Test Strategy

## Backend

- unit tests only for pure domain decisions and normalization
- controller/security tests for HTTP/auth behavior
- PostgreSQL Testcontainers for repositories, constraints, and transactions
- integration tests for state + event atomicity
- fake/WireMock provider tests for SMS boundary

## Frontend

- unit tests for schemas, utilities, and small stores
- Testing Library for forms and interaction behavior
- MSW for API-boundary component tests
- Playwright for critical user journeys

## Test data

Use builders/fixtures that make these explicit:

```txt
user timezone
current clock instant
planned local date
Task status
carry count
onboarding status
sessionEpoch
```

Avoid fixtures that quietly depend on the machine's current timezone or real clock.

---

# Risks and Mitigations

## Risk: backend waits for final UI

Mitigation:

- implement against accepted contracts
- use simple temporary UI when required
- do not block transaction/security work on visual polish

## Risk: frontend gets ahead with inaccurate mocks

Mitigation:

- generate or manually maintain contract fixtures from the accepted API spec
- review mock changes alongside API changes

## Risk: event log drifts from domain state

Mitigation:

- same transaction
- mandatory rollback tests
- no asynchronous event publishing

## Risk: timezone bugs appear late

Mitigation:

- deterministic Clock tests from Phase A
- Asia/Tehran boundary tests before Today/Reconcile UI completion

## Risk: auth consumes too much implementation time

Mitigation:

- single JWT
- fake SMS adapter first
- no refresh tokens
- no password/OAuth
- provider integration after core auth behavior passes

## Risk: design polish delays core-loop validation

Mitigation:

- define a minimum usable design gate
- polish critical friction points first: OTP, Task creation, Reconcile decisions
- defer decorative systems that do not affect comprehension or speed

## Risk: “small improvements” expand scope

Mitigation:

- every new feature must name which accepted Phase 1 requirement it satisfies
- otherwise move it to Phase 2 or archive

---

# Decisions Needed Before Formalization

Claude should review the plan and answer:

1. Is the proposed sequence dependency-correct?
2. Should database foundation be a separate milestone or remain inside the first backend slice?
3. Should schema migrations be split as proposed, or use one initial migration?
4. Should Auth be completed before onboarding/Goal/Task work begins, or can domain APIs initially use a temporary test principal?
5. Is the proposed vertical-slice structure realistic for a three-person team?
6. Is any phase too large and in need of splitting?
7. Is any phase too ceremonial and safe to merge?
8. Should initial Task creation with `plannedForDate` emit only `TASK_CREATED`, or both `TASK_CREATED` and `TASK_PLANNED`?
9. Should Goal deletion be implemented before the pilot, or could deletion be excluded from the initial validation candidate?
10. Should `logout-all` be implemented for operational support even if hidden from the UI?
11. Should measurement use direct SQL/scripts, or is a minimal admin endpoint justified?
12. Is once-per-local-day `APP_OPENED` sufficient for accepted metrics?
13. Are the release-gate checks complete?
14. Which tests are mandatory before beginning frontend integration, versus mandatory only before validation?
15. Are there missing production-readiness tasks for a small VPS deployment?
16. Should PWA work happen earlier to expose install/update issues, or remain near deployment?
17. What work can the designer complete before backend endpoints exist without creating rework?
18. Should OpenAPI be treated as generated documentation from controllers or contract-first source material?
19. What is the smallest revision required before this becomes `05-Implementation/phase-1-plan.md`?
20. Which parts are overengineered for the 10–20 tester, 14-day validation?

## Requested Review Style

Please be concrete and skeptical:

- identify dependency mistakes
- identify hidden critical-path work
- distinguish validation blockers from post-pilot improvements
- cut ceremony where it does not reduce risk
- preserve accepted scope and architecture
- propose exact sequence replacements where needed

---

# Expected Outcome

After Claude review and Mahdi approval:

1. resolve this discussion
2. formalize:

```txt
05-Implementation/phase-1-plan.md
```

3. use the formal plan as the build sequence and release-gate source of truth
4. close the Phase 0 definition/architecture phase
5. clean and archive redundant map content before implementation begins