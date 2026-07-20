# Discussion 020B — API and Frontend State Contracts

## Status

Open after Discussion 020A stabilizes.

## Scope

Define the transport-facing contracts that expose accepted Planning, Reconcile, confirmation, and deterministic mutation behavior to clients.

## Required Inputs

- final Discussion 020A runtime boundaries
- final Discussion 019A canonical model
- final Discussion 019B transaction, concurrency, and idempotency contracts
- final Discussion 019C observability and retention contracts
- accepted PlanningDraft and Reconcile semantics from Discussions 013–018

## Questions

1. Which API resources represent PlanningDraft, ReconcileSession, confirmation, and command result?
2. Which operations are synchronous, asynchronous, polled, or streamed?
3. How are draft revisions and stale drafts represented?
4. Which requests carry expected entity versions?
5. Which requests require idempotency keys?
6. How are affected entity IDs and consequence previews transferred without trusting the client?
7. How does the client submit explicit confirmation?
8. How are accepted-but-conflicted recommendations represented?
9. Which error envelope distinguishes validation, conflict, provider failure, cancellation, and authorization failure?
10. Which frontend states exist for Planning and Reconcile?
11. How is user-authored input preserved after failure?
12. How are degraded-mode FACT and RULE_MATCH responses represented without AI_EXPLANATION?
13. How are all-or-nothing bulk commands represented?
14. What is the refresh contract after stale-version conflict?
15. How are cancellation and abandoned requests distinguished from timeout or provider failure?

## Locked Inputs

- partial AI output is never an approvable client state in MVP
- client-provided consequences are not mutation authority
- confirmed mutations use deterministic command handlers
- bulk actions are all-or-nothing
- stale confirmation requires refreshed preview and new confirmation
- recommendation acceptance does not prove command success
- manual operation remains available when AI is unavailable

## Expected Decisions

- endpoint and resource families
- request/response contracts
- error envelope
- frontend state machines
- conflict and refresh contract
- confirmation and command contract
- cancellation contract
- polling/streaming transport decisions
- Mind Map impact

## Out of Scope

- provider SDK usage
- prompt construction
- schema repair internals
- final timeout and retry thresholds
- UI styling
- implementation sequencing
