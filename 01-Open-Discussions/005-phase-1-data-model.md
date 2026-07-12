# Discussion 005 — Phase 1 Data Model

## Status

Resolved and formalized in [[04-Specs/data-model-phase-1]].

## Context

This discussion reviewed the minimum data model needed for the accepted Phase 1 loop:

```txt
Create tasks
→ optionally connect them to goals
→ optionally plan them for a day
→ return after that day
→ reconcile unfinished work through Done / Carry / Drop
→ record behavioral events
```

Reference decisions:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]
- [[04-Specs/phase-1-implementation-stack]]

## Review Outcome

Claude's review was accepted with one explicit product decision from Mahdi:

```txt
Keep RECONCILE_STARTED.
```

It remains useful for distinguishing:

```txt
Reconcile shown
but user did not engage
```

from:

```txt
Reconcile shown
and user started resolving tasks
```

## Accepted Core Models

```txt
User
Goal
Task
BehavioralEvent
```

## Removed Model

### `ReconcileSession`

Removed from Phase 1.

The following can be derived from `BehavioralEvent` using `eventType`, `occurredAt`, and `localDate`:

- same-day no-retrigger checks
- shown / started / skipped / completed state
- flow duration
- return and engagement analysis

Phase 1 has only one Reconcile entry path, so a separate session entity would add lifecycle and transaction complexity without solving an existing ambiguity.

It may return in a later phase if multiple Reconcile entry points or cross-flow grouping create a real requirement.

## Accepted Model Decisions

### User

- keep `onboardingStatus` and `onboardingCompletedAt` on User
- do not create a separate onboarding progress model
- keep timezone explicit with `Asia/Tehran` as the Phase 1 default

### Goal

- remove `archivedAt`
- remove `GOAL_ARCHIVED`
- do not introduce goal lifecycle management until a real UX or metric requires it

### Task

- keep current state on Task and immutable behavior history in events
- keep `carryCount` on Task and record `carryCountAtEvent`
- preserve `plannedForDate` on completed and dropped tasks
- exclude reopening terminal tasks from Phase 1
- allow title editing only while active
- do not log generic `TASK_UPDATED`

### BehavioralEvent

- add `APP_OPENED`
- keep `RECONCILE_STARTED`
- remove `TASK_UPDATED`
- remove stored `MEANINGFUL_ACTION_AFTER_RECONCILE_SKIP`
- derive meaningful action after skip analytically from event timing
- keep `localDate` and `timezone` as first-class columns
- keep generic `entityType` and `entityId`
- store event type as varchar validated by the application
- keep non-null JSONB metadata with `{}` default

## Accepted Event Types

```txt
APP_OPENED

USER_CREATED
ONBOARDING_STARTED
ONBOARDING_COMPLETED

GOAL_CREATED

TASK_CREATED
TASK_PLANNED
TASK_UNSCHEDULED
TASK_COMPLETED
TASK_CARRIED
TASK_DROPPED

RECONCILE_SHOWN
RECONCILE_STARTED
RECONCILE_SKIPPED
RECONCILE_COMPLETED
```

## Database Enforcement

Task invariants must be enforced in PostgreSQL, not only in Spring validation.

Required checks include:

```txt
carryCount >= 0
COMPLETED implies completedAt is not null
DROPPED implies droppedAt is not null
ACTIVE implies completedAt and droppedAt are null
```

The final SQL syntax belongs to implementation and migrations, but the invariants themselves are locked in the formal spec.

## Final Position

The Phase 1 schema should remain intentionally small:

```txt
User
Goal
Task
BehavioralEvent
```

No Reconcile session table, no goal lifecycle model, no generic CRUD events, and no stored derived metric.

The formalized result is now owned by:

```txt
04-Specs/data-model-phase-1.md
```
