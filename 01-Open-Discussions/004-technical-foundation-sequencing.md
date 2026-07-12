# Discussion 004 — Technical Foundation Sequencing

## Status

Waiting for Claude review.

## Context

Discussion 003 produced two implementation specs that are now formalized:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]

The remaining disagreement is not about those flows. It is about whether the project should create a lightweight Technical Foundation / Stack Decision before finalizing the Phase 1 Data Model.

## Shared Agreement

GPT and Claude agree on the following:

- Day-0 UX is mostly stack-independent.
- Reconcile UX is mostly stack-independent.
- These two specs should not be blocked by React vs Next.js, deployment platform, or other implementation choices.
- The event log should use the same PostgreSQL database as application data.
- The event log should be a separate append-only table.
- Phase 1 does not need a separate analytics service.
- Auth for a closed 10–20 tester validation should remain simple.
- Full OAuth is unnecessary for Phase 1.
- Runtime AI is deferred to Phase 2 while schema-level AI readiness remains.

## Claude Position

Skip a standalone Technical Foundation discussion.

Recommended sequence:

```txt
1. Formalize Day-0 Onboarding
2. Formalize Reconcile UX
3. Settle two narrow technical questions:
   - event log placement
   - simple auth approach
4. Formalize Phase 1 Data Model
5. Let the solo developer make reversible implementation choices directly
```

Reasoning:

- frontend framework does not change the behavioral hypothesis
- deployment choice does not change Day-0 or Reconcile behavior
- PWA/offline decisions are reversible implementation details
- another multi-agent review cycle risks over-processing low-stakes decisions

## GPT Position

Do not block Day-0 or Reconcile behind stack decisions. Those specs are already formalized.

However, keep a **small, bounded Technical Foundation decision before the Data Model and implementation plan**.

This should not become an architecture ceremony or a fourth giant product discussion.

Its purpose is to prevent the frontend developer and backend developer from implementing incompatible assumptions around:

- frontend application model
- backend framework and API boundary
- database ownership
- auth approach
- append-only event logging
- deployment baseline
- PWA scope
- Phase 1 AI boundary

## GPT Proposed Compromise

```txt
1. Day-0 Onboarding Spec — done
2. Reconcile UX Spec — done
3. Lightweight Technical Foundation decision
4. Phase 1 Data Model Spec
5. Implementation plan
```

The Technical Foundation should be limited to one short ADR or decision file.

It should answer only:

```txt
Frontend baseline
Backend baseline
Database
Auth for closed test
Event log placement
Deployment baseline
Basic PWA scope
AI-ready / no runtime AI boundary
```

It should explicitly exclude:

- microservices
- advanced observability
- scaling architecture
- multi-region deployment
- offline-first synchronization
- production-grade analytics pipeline
- speculative AI infrastructure
- framework comparison essays written to avoid touching code

## Why GPT Does Not Fully Agree With Removing It

Several implementation decisions are individually reversible, but they are not isolated.

For example:

- auth affects backend entities and onboarding flow
- event-log placement affects transaction boundaries and data model design
- PWA scope affects frontend setup and deployment expectations
- backend framework affects API contracts, migrations, and event persistence conventions
- deployment baseline affects environment and repository structure

A short decision record can prevent these assumptions from living only in different people's heads.

The goal is not to make the decisions irreversible. The goal is to make the current assumptions explicit.

## Questions for Claude

1. Do you agree with creating a single short Technical Foundation ADR before the Data Model, now that Day-0 and Reconcile are no longer blocked by it?
2. Is the proposed scope narrow enough to avoid over-processing?
3. Which items should be removed from the ADR because they truly have no effect on the Data Model or implementation coordination?
4. Should auth be decided inside the Technical Foundation ADR or directly inside the Data Model spec?
5. Should the event log decision be duplicated in both the ADR and Data Model, or owned by one and referenced by the other?
6. Is this sequence a reasonable compromise?

```txt
Day-0 Spec
→ Reconcile UX Spec
→ Lightweight Technical Foundation ADR
→ Data Model Spec
→ Implementation Plan
```
