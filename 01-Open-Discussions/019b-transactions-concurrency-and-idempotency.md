# Discussion 019B — Transactions, Concurrency, and Idempotency

## Status

Open after Discussion 019A stabilizes.

## Scope

Define how accepted canonical mutations execute safely under concurrency.

This discussion will cover:

- Project completion and stop cascades
- stopping active Project-owned Routines
- handling already-materialized future RoutineOccurrences
- Task Drop and Restore transaction boundaries
- Goal and Project terminal child-resolution transactions
- optimistic concurrency and entity versions
- revalidate-before-commit
- stale confirmation rejection
- idempotency keys and duplicate submission protection
- retry safety
- duplicate cascade prevention
- race conditions between parent transition, Routine edit, occurrence completion, and Task mutation
- outbox or equivalent reliable event publication boundary

## Required Inputs

- final Discussion 019A schema and invariants
- Discussion 015 lifecycle semantics
- Discussion 018A confirmation contract
- Discussion 018C retry, idempotency, and failure rules

## Expected Decisions

- transaction contracts
- lock or version strategy
- conflict response semantics
- idempotency record model
- cascade ordering
- future occurrence cancellation or invalidation policy
- event publication consistency
- scenario and failure matrix
- Mind Map impact

## Deferred Until 019A Closes

Exact foreign keys, row versions, status fields, and temporal columns must come from 019A rather than being invented independently here.
