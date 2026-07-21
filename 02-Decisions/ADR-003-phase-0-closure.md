# ADR-003 — Phase 0 Definition and Architecture Closure

## Status

Superseded — this legacy closure no longer authorizes implementation.

The AI-native project baseline remains blocked on M0 and the final resolution of [[01-Open-Discussions/022-updated-mvp-implementation-plan]].

## Decision

```txt
Phase 0 — Definition & Architecture: Closed
```

The project has enough accepted product, UX, data, security, API, stack, and implementation guidance to begin coding.

Phase 0 is closed through these source documents:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]
- [[04-Specs/phase-1-implementation-stack]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/auth-phase-1]]
- [[04-Specs/api-contracts-phase-1]]
- [[05-Implementation/phase-1-plan]]

## Consequences

Implementation must not begin from the legacy Milestone 1 in [[05-Implementation/phase-1-plan]]. The current milestone sequence becomes authoritative only after Discussion 022 completes its baseline and Mind Map gates.

The accepted Phase 1 boundaries are no longer reopened informally during coding.

Any proposed change to the following requires a new bounded discussion and, when accepted, an updated spec or ADR:

```txt
Phase 1 scope
core UX behavior
data model
behavioral event contracts
authentication/session architecture
API contracts
implementation stack
release gates
```

Minor implementation details that preserve the accepted contracts do not require a new ADR.

## Source-of-Truth Priority

When older notes conflict with newer accepted documents, use this priority:

```txt
1. latest accepted ADR
2. latest accepted formal spec
3. accepted implementation plan
4. resolved discussion summary
5. research notes and historical drafts
```

Specific supersession rules:

- [[04-Specs/auth-phase-1]] supersedes older references to password recovery, refresh tokens, rotating refresh sessions, OAuth, and multiple identity providers.
- [[04-Specs/api-contracts-phase-1]] supersedes draft endpoint options and rejected API error codes.
- [[05-Implementation/phase-1-plan]] supersedes earlier sequencing assumptions and defers Goal deletion until after the pilot.
- resolved discussions preserve decision history but are not implementation authority.

## Phase 1 Scope Reminder

Phase 1 validates the reconcile-on-open loop:

```txt
OTP login
→ optional Day-0 setup
→ Goal/Task planning
→ Today
→ Reconcile unresolved prior work
→ Done / Carry / Drop / Skip
→ behavioral event measurement
```

Still excluded:

```txt
AI runtime
Weekly Review
calendar integration
routines/habits
time blocking
deadlines
refresh tokens
OAuth/password/email login
offline-first mutations
admin analytics dashboard
Goal deletion before pilot
```

## Start Condition

The next active work is:

```txt
Milestone 1 — Running Skeleton
Phase A — Repository and Engineering Foundation
Phase B — Database Foundation
```

No additional architecture phase is required before scaffolding begins.
