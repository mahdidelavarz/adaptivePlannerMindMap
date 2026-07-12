# Discussion 004 — Technical Foundation Sequencing

## Status

Resolved and closed.

## Context

Discussion 003 produced two formal implementation specs:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]

The remaining disagreement was whether a standalone Technical Foundation decision should exist before finalizing the Phase 1 Data Model.

## Initial positions

### Claude's initial position

Skip a separate stack discussion and resolve only the narrow technical questions needed by the Data Model.

Reasoning:

- Day-0 and Reconcile UX are stack-independent.
- Frontend, deployment, and PWA decisions are mostly reversible.
- Another broad review cycle could over-process low-stakes choices.

### GPT's position

Do not block Day-0 or Reconcile behind stack decisions, but create one short bounded ADR before the Data Model.

Reasoning:

- auth changes when Day-0 begins and whether a User exists first
- event persistence changes transaction boundaries for every Reconcile action
- deployment and API assumptions should not live only in different people's heads
- the goal is documentation and coordination, not architecture ceremony

## Resolution

After Day-0 and Reconcile UX were formalized, Claude updated its position and agreed to the bounded ADR.

The decisive points were:

- auth is a cross-flow decision, not merely a backend detail
- task mutations and event appends need an explicit atomicity rule
- the earlier objection to blocking UX specs no longer applies because those specs are already complete

## Agreed sequence

```txt
Day-0 Onboarding Spec — done
→ Reconcile UX Spec — done
→ Lightweight Technical Foundation ADR — done
→ Phase 1 Data Model Spec
→ Implementation Plan
```

## Agreed ADR constraints

The ADR should:

- record the chosen frontend/backend baselines instead of hosting a framework debate
- own auth flow and timing relative to onboarding
- reduce PWA scope to a yes/no Phase 1 decision
- record event-log placement, while leaving detailed schema ownership to the Data Model spec
- state the atomic mutation + event append principle
- preserve AI schema readiness while deferring runtime LLM implementation

The ADR should not include:

- framework comparison essays
- detailed event schemas
- offline-first synchronization design
- microservices or scaling architecture
- advanced observability
- speculative AI infrastructure

## Final implementation decision

Created:

```txt
02-Decisions/ADR-002-phase-1-technical-foundation.md
```

Recorded baseline:

```txt
Frontend: React + TypeScript + Vite
Backend: Java + Spring Boot
Database: PostgreSQL
Auth: OTP-only + JWT
API: versioned REST
Event log: same PostgreSQL database, separate append-only table
Transactions: domain mutation + event append are atomic
Deployment: Docker-based VPS baseline
PWA Phase 1: yes
Offline-first: deferred
Runtime AI: deferred to Phase 2
```

## Auth clarification from Mahdi

Phase 1 uses only:

```txt
OTP + JWT
```

Explicitly excluded:

- password authentication
- Google/social OAuth
- multiple parallel auth methods

A new user enters Day-0 onboarding after successful OTP verification and JWT issuance.

## Ownership boundaries

### Technical Foundation ADR owns

- stack baseline
- auth flow/timing
- API boundary
- deployment baseline
- PWA yes/no
- AI runtime boundary
- atomic write principle

### Phase 1 Data Model spec owns

- Goal / Task / Event fields
- event types and required metadata
- indexes and constraints
- detailed append-only event schema

This prevents duplicated reasoning from drifting between documents.

## Next step

Create:

```txt
04-Specs/data-model-phase-1.md
```

using [[02-Decisions/ADR-002-phase-1-technical-foundation]] as its technical baseline.