# Discussion 019 — Data Persistence and Observability Program

## Status

Split into three focused discussions after dependency audit.

This file is now the program hub for persistence work. It does not define a fourth competing data model.

Child discussions:

- [[01-Open-Discussions/019a-canonical-data-model-and-invariants]]
- [[01-Open-Discussions/019b-transactions-concurrency-and-idempotency]]
- [[01-Open-Discussions/019c-events-ai-observability-and-retention]]

---

## 1. Why the Split Exists

The original Discussion 019 combined three different decision layers:

```txt
canonical persistence truth
transaction and concurrency behavior
observability and analytics history
```

These layers depend on one another but must not be designed as one undifferentiated schema.

Accepted sequence:

```txt
019A canonical model and invariants
→ 019B transactions, concurrency, and idempotency
→ 019C events, AI observability, and retention
```

019B and 019C may be drafted in parallel only after 019A ownership, identity, lifecycle, and temporal fields are stable.

---

## 2. Resolved Contradiction from the Original Skeleton

The original skeleton incorrectly listed:

```txt
no Task → Goal
no Routine → Goal
```

That contradicts accepted Discussion 012.

Authoritative ownership is:

```txt
Project
→ Goal or standalone

Task
→ Goal or Project or standalone
→ never Goal and Project simultaneously

Routine
→ Goal or Project or standalone
→ never Goal and Project simultaneously
```

Direct Goal-owned Tasks and Routines are canonical and must be representable in persistence.

---

## 3. Program Boundaries

### 019A owns

- canonical tables and fields
- relationships and foreign keys
- lifecycle states
- temporal checkpoint fields
- placement and provenance
- exclusive-parent constraints
- identity preservation
- canonical-versus-derived field boundary
- minimum indexes required by accepted product behavior

### 019B owns

- transaction boundaries
- parent lifecycle cascades
- RoutineOccurrence shutdown behavior
- optimistic concurrency
- stale confirmation enforcement
- idempotency
- retry safety
- duplicate cascade prevention
- conflict handling

### 019C owns

- domain and audit events
- PlanningDraft persistence decision
- AI proposal and session records
- rule-match observability
- accepted, edited, rejected, and ignored suggestion history
- model and prompt versioning
- logging layers
- raw prompt/response retention
- analytics-safe summaries

---

## 4. Accepted Inputs

This program consumes authoritative decisions from:

- Discussion 012 and 012A
- Discussions 013–014A
- Discussions 015–016A
- Discussion 017B
- Discussion 018A final resolution
- Discussion 018C final resolution

It must not reopen:

- which canonical entities exist
- direct Goal ownership
- absence of a persisted canonical Plan entity
- Task/Routine exclusive-parent ownership
- user-confirmed Goal achievement
- Project-owned Routine shutdown semantics
- review due versus execution overdue
- AI proposal versus deterministic mutation authority
- privacy and free-text boundaries

---

## 5. Shared Principles

```txt
canonical tables store product truth
history stores decisions and transitions
derived values remain derived unless a justified snapshot is required
analytics must not masquerade as Goal progress
AI is never the authorizing actor for canonical mutation
```

---

## 6. Closure Order

Discussion 019 is complete only when:

1. 019A is accepted.
2. 019B is accepted against the final 019A model.
3. 019C is accepted against the final 019A and 019B contracts.
4. Mind Map impacts are consolidated.
5. Discussions 020–022 receive explicit implementation and validation dependencies.
