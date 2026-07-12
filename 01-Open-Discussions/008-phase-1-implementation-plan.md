# Discussion 008 — Phase 1 Implementation Plan

## Status

Resolved after Claude review and Mahdi approval.

## Result

The accepted implementation sequence is formalized in:

- [[05-Implementation/phase-1-plan]]

This discussion no longer owns active implementation decisions.

## Team Assumption

The plan is explicitly designed for a three-person team:

```txt
Mahdi: frontend implementation + product/spec ownership + E2E coordination
Backend developer: Java/Spring/PostgreSQL + security + transactions + deployment backend
Designer: UX/UI + interaction states + responsive behavior + usability review
```

The designer is not treated as a third engineer. The project therefore has two implementation streams and one design stream that stays slightly ahead of development.

## Accepted Review Changes

### Parallel development without waiting for full Auth

Add a development-only endpoint:

```txt
POST /api/v1/dev/test-session
```

Rules:

- enabled only in `local` and `test` profiles
- completely unavailable in `production`
- creates or loads a fixed test User
- issues the same valid JWT cookie used by the real application
- exists only to unblock frontend and domain integration while OTP work is incomplete
- must have an integration test proving production returns 404

This endpoint is development tooling, not a Phase 1 product endpoint.

### Actual execution model

The plan is not a strict single-file queue.

After Foundation and Database work establish a stable contract boundary:

```txt
Auth implementation
and
User/Goal/Task frontend + domain slices
```

may progress partially in parallel through the test-session mechanism and contract-accurate mocks.

### Migration split

Use:

```txt
V1: identity, OTP, BehavioralEvent foundation
V2: Goal and Task domain
```

Do not create one migration per table and do not place the full product schema into one opaque migration.

### Task creation event

Creating a Task that already contains `plannedForDate` appends only:

```txt
TASK_CREATED
```

The initial planning date is recorded in that event's metadata.

Do not also append `TASK_PLANNED` for the same creation command.

### Goal deletion

Goal deletion is excluded from the validation candidate.

The API contract may retain the documented future behavior, but implementation is deferred until after the pilot because no accepted Day-0, Today, or Reconcile flow requires deletion.

### Reconcile split

Split the previous Reconcile phase into:

```txt
G1: backend commands, persistence, concurrency, atomicity, and integration tests
G2: frontend interaction, Drop Undo timer, and E2E
```

G1 must pass its exit gate before G2 is considered integrated.

### Measurement split

Separate:

```txt
H1: shipped APP_OPENED behavior and event correctness
H2: validation SQL/scripts
```

H2 does not block infrastructure deployment, but the core metric queries must be implemented and verified against seeded data before the pilot starts.

### Logout-all / forced revocation

Do not build a Phase 1 UI or unnecessary public feature around logout-all.

Operational forced revocation may use:

- a small internal script or backend command
- the accepted `sessionEpoch` mechanism

Avoid undocumented ad-hoc SQL as the normal operating procedure.

### Production readiness

Add external uptime monitoring against the deployed health endpoint.

The monitoring service must notify a channel the team actively checks.

### Release gate additions

The validation may not start until:

```txt
event payload privacy review passes
OTP resend/rate-limit/max-attempt behavior is verified against production configuration
```

These are blocking checks, not optional cleanup.

### OpenAPI ownership

- the Markdown API spec remains the human-reviewed design authority
- OpenAPI is generated from Spring controllers
- do not maintain a second independent hand-written OpenAPI contract

### Designer workflow

The designer works from formal API and UX states, including:

- OTP error codes
- loading, empty, error, and success states
- Reconcile available/unavailable states
- `remainingCount`
- session-expired and session-revoked behavior

The designer may work ahead of implementation but must not invent response shapes or hidden product behavior.

### PWA timing

Keep PWA work late, after the core loop is stable.

Installability and update behavior do not block early core-loop integration.

## Final Critical Path

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
→ Infrastructure/PWA
→ validation scripts + release gate
```

## Superseded Assumptions

Rejected:

- treating the entire plan as strictly sequential
- blocking all product work until OTP provider integration is complete
- treating the designer as a third engineering stream
- building Goal deletion before the pilot
- combining backend Reconcile correctness and frontend UX in one exit gate
- letting analysis tooling block infrastructure deployment
- building an admin dashboard for 10–20 testers
- moving PWA earlier than the core loop
- maintaining OpenAPI as an independent manually edited source of truth

## Next Step

Use [[05-Implementation/phase-1-plan]] as the execution and release-gate source of truth.