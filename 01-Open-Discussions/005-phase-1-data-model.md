# Discussion 005 — Phase 1 Data Model

## Status

Waiting for Claude review.

## Context

The following decisions are already formalized:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]
- [[04-Specs/phase-1-implementation-stack]]

The next step is to define the smallest data model needed for the Phase 1 loop:

```txt
Create tasks
→ optionally connect them to goals
→ optionally plan them for a day
→ return after that day
→ reconcile unfinished work through Done / Carry / Drop
→ record behavioral events
```

This is a domain-model proposal for review. It is not yet the final SQL schema, JPA implementation, API DTO design, or migration plan.

## Accepted Constraints

- normalized Iranian phone number is the single Phase 1 account identity
- new users default to `Asia/Tehran`
- timestamps are stored in UTC
- user-facing day boundaries use the user's timezone
- `plannedForDate` is date-only
- goals are optional
- tasks may exist without goals
- tasks may remain unscheduled
- Phase 1 has no deadlines, routines, time-blocking, productivity score, or runtime AI
- behavioral events use a separate append-only table in the same PostgreSQL database
- a domain mutation and its event append must succeed atomically

---

# Proposed Models

## 1. User

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

### Notes

- `timezone` remains explicit even though Phase 1 defaults to Iran.
- Onboarding completion cannot be inferred from goal or task creation because every Day-0 creation step is optional.
- `displayName` should remain optional unless the final onboarding UI requires it.

### Open question

Should onboarding use fields on `User`, or a separate `OnboardingProgress` model?

GPT preference: keep it on `User` for Phase 1. A separate model is unjustified unless onboarding becomes a long resumable workflow.

---

## 2. Goal

```txt
Goal
- id: UUID
- userId: UUID, required
- title: String, required
- archivedAt: Instant?
- source: MANUAL | AI_GENERATED
- createdAt: Instant
- updatedAt: Instant
```

### Notes

- Goals are deliberately lightweight.
- No target date, progress percentage, hierarchy, category, or priority is proposed.
- `AI_GENERATED` preserves schema-level readiness only. Runtime AI remains excluded.
- `archivedAt` is preferred over both `status` and `archivedAt`; storing two versions of the same truth is how innocent schemas become crime scenes.

### Open question

Does Phase 1 need goal archiving at all, or can goal deletion be excluded until evidence requires lifecycle management?

---

## 3. Task

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

### Reconcile eligibility

```txt
status = ACTIVE
AND plannedForDate IS NOT NULL
AND plannedForDate < user's current local date
```

The comparison must use the user's timezone, not the server UTC date.

### Carry mutation

```txt
status remains ACTIVE
plannedForDate changes
carryCount increments by 1
```

### Done mutation

```txt
status becomes COMPLETED
completedAt is set
plannedForDate remains unchanged
```

### Drop mutation

```txt
status becomes DROPPED
droppedAt is set
plannedForDate remains unchanged
```

### Open questions

1. Should completed and dropped tasks preserve `plannedForDate`, or should only event metadata preserve it?
2. Should reopening terminal tasks exist in Phase 1?
3. Should title edits be allowed after completion or drop?
4. Should `carryCount` be stored, derived from events, or both?

GPT preference:

- preserve `plannedForDate` on terminal tasks
- exclude reopen from Phase 1
- allow title editing only while active
- store `carryCount` on Task and also record `carryCountAtEvent`

---

## 4. BehavioralEvent

```txt
BehavioralEvent
- id: UUID
- userId: UUID, required
- eventType: String, required
- entityType: USER | GOAL | TASK | RECONCILE_SESSION | null
- entityId: UUID?, optional
- occurredAt: Instant, required
- localDate: LocalDate, required
- timezone: String, required
- source: USER | SYSTEM
- metadata: JSONB, required, default {}
- schemaVersion: Integer, required, default 1
```

### Why store `localDate` and `timezone`

`occurredAt` remains the UTC source timestamp.

`localDate` and `timezone` preserve the user's day context at event time. Later analysis should not reconstruct historical local dates from a timezone setting that may have changed.

### Why JSONB metadata

Different events need different small payloads. JSONB avoids a table full of unrelated nullable columns.

Each event type must still have a documented metadata contract. JSONB is not permission to dump arbitrary application state into the event log.

### Proposed Phase 1 event types

```txt
USER_CREATED
ONBOARDING_STARTED
ONBOARDING_COMPLETED
GOAL_CREATED
GOAL_ARCHIVED
TASK_CREATED
TASK_UPDATED
TASK_PLANNED
TASK_UNSCHEDULED
TASK_COMPLETED
TASK_CARRIED
TASK_DROPPED
RECONCILE_SHOWN
RECONCILE_STARTED
RECONCILE_SKIPPED
RECONCILE_COMPLETED
MEANINGFUL_ACTION_AFTER_RECONCILE_SKIP
```

### Required metadata examples

#### `TASK_CARRIED`

```json
{
  "previousPlannedForDate": "2026-07-11",
  "newPlannedForDate": "2026-07-12",
  "carryCountAtEvent": 2
}
```

#### `TASK_DROPPED`

```json
{
  "previousPlannedForDate": "2026-07-11",
  "carryCountAtEvent": 2
}
```

#### `RECONCILE_SHOWN`

```json
{
  "unresolvedTaskCountShown": 3
}
```

#### `RECONCILE_SKIPPED`

```json
{
  "unresolvedTaskCountShown": 3
}
```

### Open questions

1. Should `localDate` and `timezone` be columns, metadata, or derived later?
2. Should `entityType/entityId` remain generic, or use explicit nullable foreign keys?
3. Should event type use a database enum, varchar with application validation, or lookup table?
4. Should `MEANINGFUL_ACTION_AFTER_RECONCILE_SKIP` be stored or derived analytically?

GPT preference:

- keep `localDate` and `timezone` as columns
- use generic `entityType/entityId`
- store event type as varchar validated by the application
- use non-null metadata with `{}` default
- derive `MEANINGFUL_ACTION_AFTER_RECONCILE_SKIP` unless real-time product behavior needs the event

---

## 5. ReconcileSession

```txt
ReconcileSession
- id: UUID
- userId: UUID, required
- status: SHOWN | STARTED | SKIPPED | COMPLETED
- localDate: LocalDate, required
- timezone: String, required
- unresolvedTaskCountShown: Integer, required
- shownAt: Instant, required
- startedAt: Instant?
- skippedAt: Instant?
- completedAt: Instant?
```

### Why it may be useful

Without a session model, events can record the flow, but grouping several Done / Carry / Drop actions into one Reconcile attempt becomes less reliable.

A session can answer:

- which decisions belonged to one Reconcile flow
- how long the flow took
- whether it was shown, started, skipped, or completed
- how many unresolved tasks were presented
- whether same-day no-retrigger has already been activated

Task events performed during Reconcile can include `reconcileSessionId` in metadata.

### Open question

Is `ReconcileSession` justified, or should Reconcile exist only through events?

GPT preference: keep it. Reconcile is the central Phase 1 behavior, and a first-class session simplifies measurement and same-day trigger rules.

Claude should remove it if the same guarantees can be achieved cleanly with materially less complexity.

---

# Relationship Summary

```txt
User 1 ─── * Goal
User 1 ─── * Task
Goal 1 ─── * Task       Task.goalId is optional
User 1 ─── * BehavioralEvent
User 1 ─── * ReconcileSession

Task * ─── 0..1 Goal
BehavioralEvent ─── optionally references one domain entity
Task events ─── may reference ReconcileSession through metadata
```

---

# Proposed Indexes

```txt
User(normalizedPhone) UNIQUE
Goal(userId, archivedAt)
Task(userId, status, plannedForDate)
Task(userId, goalId, status)
BehavioralEvent(userId, occurredAt)
BehavioralEvent(userId, eventType, occurredAt)
BehavioralEvent(entityType, entityId, occurredAt)
ReconcileSession(userId, localDate)
```

Potential Reconcile query:

```sql
SELECT *
FROM task
WHERE user_id = :userId
  AND status = 'ACTIVE'
  AND planned_for_date IS NOT NULL
  AND planned_for_date < :userLocalDate
ORDER BY planned_for_date ASC, created_at ASC;
```

Claude should assess whether same-day Reconcile presentation needs a unique database constraint, application transaction logic, or only a session lookup.

---

# Transaction Boundaries

These operations must be atomic:

```txt
create task + append TASK_CREATED
plan task + append TASK_PLANNED
complete task + append TASK_COMPLETED
carry task + increment carryCount + append TASK_CARRIED
drop task + append TASK_DROPPED
skip reconcile + update ReconcileSession + append RECONCILE_SKIPPED
complete reconcile + update ReconcileSession + append RECONCILE_COMPLETED
```

Do not publish events asynchronously in Phase 1. The behavioral record is part of the product mutation, not a best-effort analytics side effect.

---

# Deliberately Excluded Models

- Project
- Subtask hierarchy
- Routine or habit
- Calendar event
- Time block
- Deadline
- Priority
- Tag or category
- Energy score
- Mood log
- Productivity score
- AI recommendation
- AI conversation
- Prompt or model configuration
- Notification schedule
- Team or workspace
- Separate analytics warehouse

These may become useful later. Future usefulness is not evidence that they belong in the first schema.

---

# Questions for Claude

Please review this as a Phase 1 domain-model proposal, not finished implementation code.

1. Which models are actually required for the accepted Day-0 and Reconcile flows?
2. Is `ReconcileSession` justified, or should Reconcile use only events?
3. Is storing both Task state and events the correct tradeoff?
4. Should `carryCount` be stored, derived, or both?
5. Should terminal tasks preserve `plannedForDate`?
6. Is `onboardingStatus` on User sufficient?
7. Does Goal need an archive lifecycle in Phase 1?
8. Are `localDate` and `timezone` necessary event columns?
9. Should generic `entityType/entityId` be replaced with explicit nullable foreign keys?
10. Which events are missing, redundant, or too implementation-oriented?
11. Should `MEANINGFUL_ACTION_AFTER_RECONCILE_SKIP` be stored, derived, or both?
12. Which invariants and constraints should be enforced by PostgreSQL rather than only by Spring?
13. What is the smallest revision needed before this becomes `04-Specs/data-model-phase-1.md`?

## Requested Review Style

Please be critical and concrete:

- identify unnecessary complexity
- identify missing invariants
- distinguish required Phase 1 fields from future-friendly speculation
- propose exact replacements, not only objections
- preserve accepted product decisions unless a contradiction is found
