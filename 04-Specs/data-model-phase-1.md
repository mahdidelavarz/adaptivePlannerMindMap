# Phase 1 Data Model

## Status

Accepted for Phase 1.

Formalized from [[01-Open-Discussions/005-phase-1-data-model]] after Claude review and Mahdi's decision to retain `RECONCILE_STARTED`.

## Purpose

Define the minimum persistent domain model required to implement and measure the Phase 1 loop:

```txt
Create tasks
→ optionally connect them to goals
→ optionally plan them for a day
→ return after that day
→ reconcile unfinished work through Done / Carry / Drop
→ record behavioral events
```

This spec owns domain entities, relationships, invariants, event contracts, indexes, and transaction boundaries.

It does not define final Java classes, API DTOs, migration filenames, controller structure, or exact SQL syntax.

## Technical Baseline

Use this spec with:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]
- [[04-Specs/phase-1-implementation-stack]]

Accepted technical rules:

- PostgreSQL stores both application state and behavioral events
- behavioral events use a separate append-only table
- timestamps are stored as UTC instants
- user-facing calendar days use the user's IANA timezone
- new Phase 1 users default to `Asia/Tehran`
- `plannedForDate` is date-only
- state mutation and corresponding event append are atomic

---

# Core Models

## User

```txt
User
- id: UUID
- normalizedPhone: String, unique, required
- displayName: String?, optional
- timezone: String, required, default "Asia/Tehran"
- onboardingStatus: NOT_STARTED | IN_PROGRESS | COMPLETED
- onboardingCompletedAt: Instant?
- createdAt: Instant
- updatedAt: Instant
```

### Rules

- `normalizedPhone` is the single Phase 1 identity anchor.
- `timezone` must contain an IANA timezone identifier, never a fixed offset.
- onboarding completion must not be inferred from goal or task existence because all Day-0 creation steps are optional.
- no separate onboarding progress model is required in Phase 1.

---

## Goal

```txt
Goal
- id: UUID
- userId: UUID, required
- title: String, required
- source: MANUAL | AI_GENERATED
- createdAt: Instant
- updatedAt: Instant
```

### Rules

- goals are optional
- a user may create tasks without creating a goal
- Phase 1 does not implement goal archive, status, target date, progress percentage, hierarchy, category, or priority
- `AI_GENERATED` is schema-level readiness only; runtime AI remains excluded
- goal deletion behavior must be decided during implementation, but it must not delete historical behavioral events

---

## Task

```txt
Task
- id: UUID
- userId: UUID, required
- goalId: UUID?, optional
- title: String, required
- status: ACTIVE | COMPLETED | DROPPED
- plannedForDate: LocalDate?, optional
- carryCount: Integer, required, default 0
- source: MANUAL | AI_GENERATED
- completedAt: Instant?
- droppedAt: Instant?
- createdAt: Instant
- updatedAt: Instant
```

### Required invariants

```txt
goalId may be null
plannedForDate may be null
carryCount >= 0
COMPLETED implies completedAt is not null
DROPPED implies droppedAt is not null
ACTIVE implies completedAt and droppedAt are null
```

These invariants must be enforced through PostgreSQL constraints in addition to Spring validation.

### Reconcile eligibility

A task is eligible for Reconcile when:

```txt
status = ACTIVE
AND plannedForDate IS NOT NULL
AND plannedForDate < user's current local date
```

The backend must calculate the user's current local date from `User.timezone`. Server UTC date must not be used as a substitute.

### Carry

```txt
status remains ACTIVE
plannedForDate changes to selected date
carryCount increments by 1
```

The mutation appends `TASK_CARRIED` atomically.

### Done

```txt
status becomes COMPLETED
completedAt is set
plannedForDate remains unchanged
```

Preserving `plannedForDate` retains the historical planning context without requiring an event join for basic task inspection.

### Drop

```txt
status becomes DROPPED
droppedAt is set
plannedForDate remains unchanged
```

### Editing and reopening

- title editing is allowed only while `status = ACTIVE`
- no generic `TASK_UPDATED` event is recorded
- completed and dropped tasks cannot be reopened in Phase 1

---

## BehavioralEvent

```txt
BehavioralEvent
- id: UUID
- userId: UUID, required
- eventType: String, required
- entityType: USER | GOAL | TASK | null
- entityId: UUID?, optional
- occurredAt: Instant, required
- localDate: LocalDate, required
- timezone: String, required
- source: USER | SYSTEM
- metadata: JSONB, required, default {}
- schemaVersion: Integer, required, default 1
```

### Append-only rule

Behavioral events are immutable after insertion.

Corrections must be represented through later events or implementation-level remediation, not by casually rewriting history because reality became inconvenient.

### Historical day context

`occurredAt` stores the UTC instant.

`localDate` and `timezone` store the user's day context at event time. They must not be derived later from the user's current timezone because a timezone change would reinterpret historical behavior.

### Entity references

Use generic `entityType` and `entityId` without database foreign keys.

Reason:

- one table references multiple entity types
- events must survive entity deletion
- permanent behavioral history must not be blocked or cascaded by domain lifecycle operations

### Event type storage

Store `eventType` as varchar validated by the application.

Do not use a PostgreSQL enum that requires a migration for every new event type.

### Metadata

`metadata` is non-null JSONB with `{}` as its default.

Each event type must have a documented metadata contract. Arbitrary state dumps are not permitted.

---

# Phase 1 Event Types

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

## Why `APP_OPENED` is required

It supports:

- D1 / D3 / D7 retention
- return-after-absence analysis
- comparison between opening, seeing Reconcile, starting it, and completing it

## Why `RECONCILE_STARTED` remains

It distinguishes:

```txt
Reconcile was shown but the user did not engage
```

from:

```txt
Reconcile was shown and the user began resolving work
```

This distinction may be useful later even if Phase 1 initially uses only a subset of the available analysis.

## Derived metric

Do not store:

```txt
MEANINGFUL_ACTION_AFTER_RECONCILE_SKIP
```

Derive it from a `RECONCILE_SKIPPED` event followed within five minutes by one of:

```txt
TASK_CREATED
TASK_PLANNED
TASK_COMPLETED
TASK_CARRIED
TASK_DROPPED
RECONCILE_COMPLETED
```

If weekly review is added to the Phase 1 implementation, its completion event may also qualify.

---

# Required Event Metadata

## `RECONCILE_SHOWN`

```json
{
  "unresolvedTaskCountShown": 3
}
```

## `RECONCILE_SKIPPED`

```json
{
  "unresolvedTaskCountShown": 3
}
```

## `TASK_CARRIED`

```json
{
  "previousPlannedForDate": "2026-07-11",
  "newPlannedForDate": "2026-07-12",
  "carryCountAtEvent": 2
}
```

## `TASK_DROPPED`

```json
{
  "previousPlannedForDate": "2026-07-11",
  "carryCountAtEvent": 2
}
```

## `TASK_COMPLETED`

When completion occurs during Reconcile, metadata should include enough context to identify the previous planned day:

```json
{
  "previousPlannedForDate": "2026-07-11",
  "carryCountAtEvent": 1
}
```

The implementation may omit fields that are not applicable, but metadata contracts must remain consistent per event type.

---

# Reconcile Without a Session Model

Phase 1 does not persist `ReconcileSession`.

Use behavioral events to determine:

- whether Reconcile was shown on the user's current local date
- whether it was started
- whether it was skipped or completed
- duration between shown, started, skipped, and completed events
- whether same-day re-trigger is prohibited

Example no-retrigger lookup:

```sql
SELECT 1
FROM behavioral_event
WHERE user_id = :userId
  AND event_type IN ('RECONCILE_SHOWN', 'RECONCILE_SKIPPED', 'RECONCILE_COMPLETED')
  AND local_date = :userLocalDate
LIMIT 1;
```

The exact query may change, but the rule is fixed:

```txt
After Reconcile has been shown for a local calendar day, do not show it again that day.
```

A session model may be reconsidered only if later phases introduce multiple entry points, overlapping attempts, or a real grouping ambiguity.

---

# Relationships

```txt
User 1 ─── * Goal
User 1 ─── * Task
Goal 1 ─── * Task       Task.goalId is optional
User 1 ─── * BehavioralEvent

Task * ─── 0..1 Goal
BehavioralEvent ─── optionally references one domain entity
```

Ownership rules:

- every Goal belongs to exactly one User
- every Task belongs to exactly one User
- a Task may reference only a Goal owned by the same User
- every BehavioralEvent belongs to exactly one User

The same-user Goal/Task ownership rule must be enforced in application transaction logic. A plain foreign key on `goalId` alone does not prove ownership equality.

---

# Required Indexes

```txt
User(normalizedPhone) UNIQUE
Goal(userId, createdAt)
Task(userId, status, plannedForDate)
Task(userId, goalId, status)
BehavioralEvent(userId, occurredAt)
BehavioralEvent(userId, eventType, occurredAt)
BehavioralEvent(userId, localDate, eventType)
BehavioralEvent(entityType, entityId, occurredAt)
```

Primary Reconcile task query:

```sql
SELECT *
FROM task
WHERE user_id = :userId
  AND status = 'ACTIVE'
  AND planned_for_date IS NOT NULL
  AND planned_for_date < :userLocalDate
ORDER BY planned_for_date ASC, created_at ASC;
```

---

# Database Constraints

The implementation must include equivalents of these checks:

```sql
CHECK (carry_count >= 0)
```

```sql
CHECK (status <> 'COMPLETED' OR completed_at IS NOT NULL)
```

```sql
CHECK (status <> 'DROPPED' OR dropped_at IS NOT NULL)
```

```sql
CHECK (
  status <> 'ACTIVE'
  OR (completed_at IS NULL AND dropped_at IS NULL)
)
```

The implementation should also prevent contradictory terminal timestamps, such as a completed task also carrying a non-null `droppedAt`.

---

# Atomic Transaction Boundaries

The following state changes and events must commit or roll back together:

```txt
create task + TASK_CREATED
plan task + TASK_PLANNED
unschedule task + TASK_UNSCHEDULED
complete task + TASK_COMPLETED
carry task + increment carryCount + TASK_CARRIED
drop task + TASK_DROPPED
mark onboarding started + ONBOARDING_STARTED
mark onboarding completed + ONBOARDING_COMPLETED
create goal + GOAL_CREATED
```

Reconcile lifecycle events that do not mutate a domain entity should still be inserted transactionally within their application command.

Do not publish behavioral events asynchronously in Phase 1. The event record is part of the accepted product behavior, not optional analytics exhaust.

---

# Explicit Exclusions

Phase 1 does not include:

- ReconcileSession
- OnboardingProgress entity
- goal archive or status
- Project
- subtask hierarchy
- routine or habit
- calendar event
- time block
- deadline
- priority
- tag or category
- energy score
- mood log
- productivity score
- AI recommendation
- AI conversation
- prompt or model configuration
- notification schedule
- team or workspace
- separate analytics warehouse
- generic CRUD event such as `TASK_UPDATED`
- stored derived event such as `MEANINGFUL_ACTION_AFTER_RECONCILE_SKIP`

---

# Implementation Handoff

The implementation plan should now define:

- PostgreSQL table and column names
- Flyway migrations
- JPA entities and converters
- Spring transaction boundaries
- event metadata validation
- repositories and Reconcile queries
- API request/response contracts
- tests for timezone day boundaries
- tests for every database invariant
- retention and Reconcile analytics queries
