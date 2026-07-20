# Discussion 019A — Canonical Data Model and Invariants

## Status

Open for GPT × Claude review.

This document is Part A of the Discussion 019 persistence program.

Companion discussions:

- [[01-Open-Discussions/019-data-model-and-ai-observability]]
- [[01-Open-Discussions/019b-transactions-concurrency-and-idempotency]]
- [[01-Open-Discussions/019c-events-ai-observability-and-retention]]

Accepted dependencies:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/012a-temporal-checkpoint-amendment]]
- [[01-Open-Discussions/014-ai-planning-output-contract]]
- [[01-Open-Discussions/014a-temporal-checkpoint-planning-draft-amendment]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/015a-temporal-checkpoint-execution-amendment]]
- [[01-Open-Discussions/017b-final-reconcile-intelligence-resolution]]
- [[01-Open-Discussions/018a-final-action-permission-trust-and-reversibility-resolution]]
- [[01-Open-Discussions/018c-final-failure-privacy-domain-and-hostile-input-resolution]]

---

## 1. Scope

This discussion defines the minimum canonical persistence model for:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

It also defines:

- identity fields
- ownership foreign keys
- exclusive-parent constraints
- lifecycle status fields
- temporal fields and provenance
- Task placement
- Routine recurrence storage boundary
- occurrence identity and historical preservation
- source/provenance fields
- canonical-versus-derived field boundaries
- minimum query indexes
- data-model implications of Drop and Restore

---

## 2. Explicitly Out of Scope

This discussion does not finalize:

- transaction isolation
- cascade transaction ordering
- optimistic-concurrency conflict UX
- idempotency storage
- domain-event taxonomy
- AI operation/session tables
- raw prompt retention
- endpoint DTOs
- provider/runtime selection
- physical SQL syntax for a particular database
- exact recurrence library or serialized recurrence format

Those belong to Discussions 019B, 019C, and 020.

---

## 3. Governing Persistence Principles

```txt
canonical tables store current product truth
history/events preserve meaningful decisions and transitions
derived facts are not duplicated as mutable truth
AI provenance never grants mutation authority
identity survives ordinary lifecycle transitions
```

No persistence shortcut may silently change the accepted meaning of Goal, Project, Task, Routine, or RoutineOccurrence.

---

## 4. Resolved Ownership Contradiction

The original Discussion 019 skeleton incorrectly stated:

```txt
no Task → Goal
no Routine → Goal
```

Discussion 012 is authoritative and accepts direct Goal ownership.

Final conceptual relationships:

```txt
Goal
├── Projects
├── direct Tasks
└── direct Routines

Project
├── Tasks
└── Project-specific Routines

Standalone
├── Project
├── Task
└── Routine
```

Persistence must support:

```txt
Project.goalId nullable
Task.goalId nullable
Task.projectId nullable
Routine.goalId nullable
Routine.projectId nullable
```

Exclusive-parent constraints:

```txt
Task:
NOT (goalId IS NOT NULL AND projectId IS NOT NULL)

Routine:
NOT (goalId IS NOT NULL AND projectId IS NOT NULL)
```

Both null means standalone.

If a Task or Routine belongs to a Project, Goal context is derived through `Project.goalId`; the child does not duplicate that Goal ID.

---

## 5. Shared Field Conventions

Every canonical entity should include:

```txt
id
userId
createdAt
updatedAt
source
version
```

### 5.1 IDs

- IDs are stable opaque identifiers.
- ordinary lifecycle transitions do not replace identity.
- copying or continuing an entity creates a new ID only when the accepted lifecycle explicitly requires a new entity, such as Routine continuation after Stop.

### 5.2 User Ownership

Every canonical row belongs to exactly one product user or workspace boundary.

All referenced parent rows must belong to the same user/workspace.

Cross-user ownership is invalid even if a foreign-key identifier exists.

The exact authorization architecture is deferred, but the invariant is not.

### 5.3 Source

Proposed enum:

```txt
MANUAL
AI_ASSISTED
SYSTEM_MIGRATED
```

Meaning:

- `MANUAL`: created directly through a manual product flow.
- `AI_ASSISTED`: created from an approved AI proposal.
- `SYSTEM_MIGRATED`: created by an explicit migration/backfill process.

`source` records origin, not current ownership, authority, truth quality, or actor for later mutations.

An AI-assisted entity is still user-approved canonical data.

### 5.4 Version

Each mutable canonical entity carries a monotonically advancing version or equivalent concurrency token.

The field supports Discussion 018A revalidate-before-commit and Discussion 019B concurrency rules.

Exact implementation may be integer, row version, or database-native equivalent.

---

## 6. Goal Model

Proposed logical fields:

```txt
Goal
- id
- userId
- title
- desiredOutcome
- status: ACTIVE | ACHIEVED | ABANDONED
- targetDate?
- reviewDate
- reviewDateSource: USER | SYSTEM_DEFAULT | MIGRATED_DEFAULT
- lastContinuationDecisionAt?
- createdAt
- updatedAt
- terminalAt?
- source
- version
```

### 6.1 Invariants

- `desiredOutcome` is required for an approved Goal.
- an ACTIVE Goal has a valid `reviewDate`.
- `reviewDate` may not exceed an earlier `targetDate` when the review date was system-defaulted.
- `ACHIEVED` and `ABANDONED` are terminal lifecycle states.
- execution totals do not update Goal status automatically.
- Goal achievement remains user-confirmed.

### 6.2 Continuation Tracking

`lastContinuationDecisionAt` supports the per-Goal automatic-prompt suppression window from Discussion 017B.

Open design question:

- persist this convenience timestamp on Goal, or derive it from immutable continuation events in 019C?

Current proposal:

```txt
persist lastContinuationDecisionAt as a current scheduling aid
preserve the full decision history as events
```

This is a justified snapshot, not the source of historical truth.

### 6.3 Derived Goal Values

Do not persist as canonical mutable truth:

- universal progress percentage
- inferred achievement probability
- motivation score
- completion-derived Goal success
- `nextTemporalCheckpoint`

`nextTemporalCheckpoint` is derived from current accepted temporal fields.

---

## 7. Project Model

Proposed logical fields:

```txt
Project
- id
- userId
- goalId?
- title
- completionMeaning?
- status: ACTIVE | COMPLETED | STOPPED
- targetDate?
- reviewDate?
- reviewDateSource?: USER | SYSTEM_DEFAULT | MIGRATED_DEFAULT
- createdAt
- updatedAt
- terminalAt?
- source
- version
```

### 7.1 Ownership

- `goalId` may be null for standalone Project.
- when non-null, the Goal must belong to the same user/workspace.
- a Project belongs to at most one Goal.

### 7.2 Temporal Invariant

An ACTIVE Project must have at least one temporal checkpoint:

```txt
targetDate IS NOT NULL
OR reviewDate IS NOT NULL
```

A system-default review date may not be later than an earlier target date.

### 7.3 Lifecycle

```txt
ACTIVE → COMPLETED | STOPPED
```

Terminal Project status does not automatically change parent Goal status.

Project terminal transitions and child Routine shutdown are transaction concerns for 019B, but the data model must make the relationship queryable and enforceable.

---

## 8. Task Model

Proposed logical fields:

```txt
Task
- id
- userId
- goalId?
- projectId?
- title
- description?
- status: ACTIVE | COMPLETED | DROPPED
- placement: SCHEDULED | BACKLOG
- plannedDate?
- reviewDate?
- reviewDateSource?: USER | SYSTEM_DEFAULT | MIGRATED_DEFAULT
- deadline?
- isProtected
- protectionReasonCode?
- createdAt
- updatedAt
- terminalAt?
- source
- version
```

### 8.1 Exclusive Parent

Valid ownership states:

```txt
goalId != null, projectId = null
projectId != null, goalId = null
goalId = null, projectId = null
```

Invalid:

```txt
goalId != null AND projectId != null
```

### 8.2 Placement Invariants

Proposed canonical relationship:

```txt
placement = SCHEDULED
→ plannedDate IS NOT NULL

placement = BACKLOG
→ plannedDate IS NULL
→ reviewDate IS NOT NULL while ACTIVE
```

For ACTIVE Task:

```txt
plannedDate IS NOT NULL
OR reviewDate IS NOT NULL
```

A deadline does not substitute for execution placement or review checkpoint because it describes a boundary, not the product's next management action.

### 8.3 Status and Placement

When Task becomes COMPLETED or DROPPED:

- historical dates remain available.
- placement may remain as the last accepted placement snapshot.
- ordinary active-work queries exclude terminal status.

Open design question:

Should a terminal Task retain `placement` as its last state, or should placement become nullable outside ACTIVE?

Current proposal:

```txt
retain last placement and dates
status determines active visibility
history/events explain the transition
```

This avoids destroying useful state and simplifies Restore preview.

### 8.4 Drop and Restore

Task Drop is not physical deletion.

```txt
ACTIVE → DROPPED
```

Restore:

```txt
DROPPED → ACTIVE
same Task ID
new valid current temporal checkpoint required
old past plannedDate cannot silently become current commitment
```

The Task row preserves identity. Drop and Restore history belongs to events in 019C.

### 8.5 Protection

Current proposal includes:

```txt
isProtected boolean
protectionReasonCode nullable
```

Questions for review:

- should protection be a single current flag plus history events?
- should multiple simultaneous protection reasons require a child table?

MVP proposal:

```txt
one current protection flag
one primary reason code
full reason history and matched constraints in events
```

Protection may not be inferred by AI from free text.

### 8.6 Carry

Carry is not a status and should not be stored as a mutable `carryCount` source of truth on Task.

Current proposal:

```txt
Carry actions are historical events
carryCount is derived for rule evaluation
```

A cached summary may be added later only if provenance and rebuilding are defined.

---

## 9. Routine Model

Proposed logical fields:

```txt
Routine
- id
- userId
- goalId?
- projectId?
- title
- description?
- status: ACTIVE | STOPPED
- recurrenceDefinition
- recurrenceTimezone
- effectiveFromLocalDate
- effectiveUntilLocalDate?
- continuationOfRoutineId?
- createdAt
- updatedAt
- stoppedAt?
- source
- version
```

### 9.1 Exclusive Parent

Routine follows the same exclusive-parent rule as Task:

```txt
Goal or Project or standalone
never Goal and Project simultaneously
```

A Project-owned Routine belongs to exactly one Project and is not shared or reused across Projects.

### 9.2 Recurrence

`recurrenceDefinition` is canonical but its exact serialized format is deferred to runtime architecture.

It must be:

- versionable
- deterministic
- timezone-aware
- capable of deriving future occurrences
- stable enough to preserve historical interpretation

`recurrenceTimezone` is required because local-date occurrence semantics cannot safely depend on a user's future account timezone alone.

### 9.3 Stop and Continuation

```txt
ACTIVE → STOPPED
```

A stopped Routine remains historical.

Later resumption creates a new Routine:

```txt
newRoutine.continuationOfRoutineId = stoppedRoutine.id
```

Open question:

Should `continuationOfRoutineId` point only to the immediate predecessor or the original root?

Current proposal:

- store immediate predecessor.
- derive the continuation chain recursively.
- optionally add a derived/root snapshot only if performance requires it later.

### 9.4 Routine Review Date

No generic `reviewDate` is required solely for temporal visibility when recurrence can derive the next occurrence.

Routine mismatch evidence belongs to occurrence-derived analytics and events, not a hidden psychological or lifecycle field.

---

## 10. RoutineOccurrence Model

Proposed logical fields:

```txt
RoutineOccurrence
- id
- userId
- routineId
- scheduledLocalDate
- scheduledTimezone
- status: PENDING | DONE | MISSED
- statusSource: USER | SYSTEM_DETERMINISTIC | MIGRATED
- completedAt?
- resolvedAt?
- createdAt
- updatedAt
- version
```

### 10.1 Identity and Uniqueness

Every occurrence belongs to exactly one Routine.

Proposed uniqueness:

```txt
UNIQUE(routineId, scheduledLocalDate)
```

This assumes at most one occurrence per Routine per local date.

Claude should review whether the accepted recurrence model may require multiple same-day occurrences. If yes, the uniqueness key must include a stable occurrence slot/key rather than silently merging them.

### 10.2 Historical Preservation

RoutineOccurrence is historical execution evidence.

- parent Routine edits do not rewrite old occurrences.
- stopping a Routine does not delete prior occurrences.
- correction is an auditable transition, not destructive overwrite without history.
- future materialized occurrences after Stop require 019B policy.

### 10.3 Status

```txt
PENDING → DONE | MISSED
```

Corrections may change an already resolved occurrence only through an explicit correction path preserved in history.

### 10.4 User ID Duplication

`userId` may be technically derivable through Routine.

Current proposal still stores it for:

- tenant isolation
- efficient scoped queries
- defense-in-depth consistency checks

The database or transaction layer must ensure `RoutineOccurrence.userId = Routine.userId`.

Claude should review whether this justified denormalization is worth the consistency burden.

---

## 11. PlanningDraft Boundary

Accepted product rule:

```txt
Plan is not a canonical domain entity.
```

This does not automatically require every PlanningDraft to be memory-only.

Current proposal:

- no approved canonical `Plan` table exists.
- an unapproved PlanningDraft may be persisted temporarily as an operational proposal in 019C.
- draft persistence must not create Goal, Project, Task, or Routine rows before approval.
- draft rows, if used, are not canonical execution truth.

This supports crash recovery, stale validation, multi-device review, and proposal audit without resurrecting Plan as a domain entity.

---

## 12. Provenance Boundary

Canonical entity `source` records creation origin only.

Do not add provider-specific fields to Goal, Project, Task, Routine, or RoutineOccurrence.

Forbidden examples:

```txt
openAiModelName on Task
promptText on Goal
aiConfidence on Project
```

AI operation, prompt-template, rule, and model versions belong to proposal/audit records in 019C.

Canonical mutation authority is never inferred from `source = AI_ASSISTED`.

---

## 13. Canonical Versus Derived Values

### Persist canonical current truth

- identity
- ownership
- lifecycle status
- accepted dates and recurrence
- Task placement
- current protection state
- temporal provenance
- creation source
- concurrency version

### Preserve as events/history

- Carry actions
- Drop and Restore decisions
- review deferrals
- intent reaffirmations
- lifecycle transitions
- corrections
- AI proposal acceptance/rejection
- matched rule evidence

### Derive rather than persist as mutable truth

- Today membership
- execution overdue
- review due
- next temporal checkpoint
- carry count
- missed ratio
- evidence quality
- Goal progress percentage
- Goal achievement inference
- Reconcile severity
- rule-match eligibility

A derived value may later receive a rebuildable cache, but the cache must not become an independent truth source.

---

## 14. Minimum Constraints

Logical constraints required regardless of database vendor:

### Goal

```txt
ACTIVE → reviewDate IS NOT NULL
```

### Project

```txt
ACTIVE → targetDate IS NOT NULL OR reviewDate IS NOT NULL
```

### Task

```txt
NOT(goalId IS NOT NULL AND projectId IS NOT NULL)

ACTIVE → plannedDate IS NOT NULL OR reviewDate IS NOT NULL

placement = SCHEDULED → plannedDate IS NOT NULL
placement = BACKLOG → plannedDate IS NULL
ACTIVE AND placement = BACKLOG → reviewDate IS NOT NULL
```

### Routine

```txt
NOT(goalId IS NOT NULL AND projectId IS NOT NULL)
ACTIVE → valid recurrenceDefinition and recurrenceTimezone
```

### RoutineOccurrence

```txt
routineId IS NOT NULL
status = DONE → completedAt IS NOT NULL
```

Some lifecycle-dependent constraints may require transaction-layer enforcement when SQL CHECK constraints cannot safely reference parent state.

---

## 15. Parent-State Consistency

The model should prevent new active children from being attached beneath terminal parents.

Examples:

- no new ACTIVE Task under COMPLETED or STOPPED Project.
- no new ACTIVE Routine under COMPLETED or STOPPED Project.
- no new active child under ACHIEVED or ABANDONED Goal.

Because this requires cross-row state checks, it may not be expressible as a simple row CHECK constraint.

019B must define transaction-layer validation and race protection.

Existing child resolution during parent terminal transitions remains governed by Discussion 015 and 019B.

---

## 16. Delete Behavior

Ordinary product lifecycle uses terminal states, not physical deletion.

Physical deletion belongs only to explicit privacy/account deletion flows.

Foreign-key delete defaults should therefore be restrictive for ordinary operations.

Current proposal:

```txt
canonical parent deletion
→ RESTRICT by default
```

Privacy deletion may use a separate controlled process with legal/security requirements and is not an AI action.

Do not use `ON DELETE CASCADE` as a substitute for lifecycle semantics.

---

## 17. Minimum Query Indexes

Logical indexes required by accepted product flows:

### Goal

- `(userId, status)`
- `(userId, reviewDate, status)`

### Project

- `(userId, goalId, status)`
- `(userId, reviewDate, status)`
- `(userId, targetDate, status)`

### Task

- `(userId, status, plannedDate)` for Today/overdue
- `(userId, status, reviewDate)` for review due
- `(userId, projectId, status)`
- `(userId, goalId, status)`
- `(userId, placement, status)`
- `(userId, deadline, status)`

### Routine

- `(userId, status)`
- `(userId, projectId, status)`
- `(userId, goalId, status)`

### RoutineOccurrence

- unique `(routineId, scheduledLocalDate)` or accepted occurrence key
- `(userId, scheduledLocalDate, status)`
- `(routineId, scheduledLocalDate)`

Exact physical index order and partial indexes belong to implementation design.

---

## 18. Initial Logical ERD

```txt
User
├── Goal
│   ├── Project
│   │   ├── Task
│   │   └── Routine
│   │       └── RoutineOccurrence
│   ├── direct Task
│   └── direct Routine
│       └── RoutineOccurrence
├── standalone Project
│   ├── Task
│   └── Routine
│       └── RoutineOccurrence
├── standalone Task
└── standalone Routine
    └── RoutineOccurrence
```

Relational form:

```txt
Goal 1 ── 0..* Project
Goal 1 ── 0..* direct Task
Goal 1 ── 0..* direct Routine
Project 1 ── 0..* Task
Project 1 ── 0..* Routine
Routine 1 ── 0..* RoutineOccurrence
```

Task and Routine use nullable exclusive parent foreign keys.

---

## 19. Open Questions for Claude

### Q1. Is the nullable exclusive-parent model correct?

Current proposal:

```txt
Task(goalId?, projectId?)
Routine(goalId?, projectId?)
CHECK not both non-null
```

Review whether a polymorphic parent table would be safer or whether it would add unnecessary abstraction and weaken foreign-key clarity.

### Q2. Should Task placement remain populated after terminal status?

Current proposal: preserve the last placement and dates; status controls active visibility.

Review Restore, analytics, and accidental resurfacing risks.

### Q3. Is `lastContinuationDecisionAt` a justified Goal snapshot?

Alternative: derive every suppression decision from events only.

Review correctness versus query complexity and rebuildability.

### Q4. Is one protection flag and one primary reason enough for MVP?

Alternative: normalized many-reason protection records.

Review whether multiple simultaneous protections are product-critical now.

### Q5. Should Carry count remain fully derived?

Current proposal: yes; Carry actions live in history and `carryCount` is calculated.

Review rule-performance needs and risk of stale duplicated counters.

### Q6. Does Routine need `effectiveFromLocalDate` and `effectiveUntilLocalDate`?

Review whether recurrence plus created/stopped timestamps is sufficient, especially across timezone and backfill behavior.

### Q7. Is immediate-predecessor `continuationOfRoutineId` sufficient?

Review chain traversal, cycle prevention, and historical identity.

### Q8. Can a Routine have more than one occurrence per local date?

If yes, `UNIQUE(routineId, scheduledLocalDate)` is unsafe and an occurrence slot/key is required.

### Q9. Should RoutineOccurrence duplicate userId?

Review tenant isolation/query value against consistency burden.

### Q10. Are review-date provenance enums complete?

Current proposal:

```txt
USER | SYSTEM_DEFAULT | MIGRATED_DEFAULT
```

Review whether AI-assisted user-approved values need a separate source or whether entity `source` plus proposal history is sufficient.

### Q11. Should unapproved PlanningDrafts be temporarily persisted?

Current proposal: allowed as operational proposals, never as canonical Plan or entity rows.

Review privacy, stale state, multi-device UX, and complexity.

### Q12. Are any accepted temporal invariants missing?

Focus on target/deadline capping, active Backlog Tasks, Goal review requirements, and Routine recurrence.

### Q13. Are lifecycle timestamps sufficient?

Current fields use generic `terminalAt` for Goal/Project/Task and `stoppedAt` for Routine.

Review whether separate `completedAt`, `droppedAt`, `achievedAt`, and `abandonedAt` fields are preferable to events plus one terminal timestamp.

### Q14. Should canonical tables include soft-delete fields?

Current proposal: no ordinary soft-delete field; lifecycle status and explicit privacy deletion are separate concepts.

Review operational recovery and accidental deletion concerns.

### Q15. Are the minimum indexes aligned with Today, Reconcile, hierarchy, and temporal checkpoint queries?

Do not optimize for hypothetical scale, but identify missing correctness-critical access paths.

---

## 20. Scenario Checks

### Scenario A — Direct Goal Task

Expected:

- `Task.goalId` set.
- `Task.projectId` null.
- Task appears under Goal without artificial Project.

### Scenario B — Project Task inherits Goal context

Expected:

- `Task.projectId` set.
- `Task.goalId` null.
- Goal context derives through Project.

### Scenario C — Both Task parents supplied

Expected:

- validation and persistence reject the row.
- no silent parent preference.

### Scenario D — Backlog Task

Expected:

- ACTIVE.
- `placement = BACKLOG`.
- `plannedDate = null`.
- future `reviewDate` exists.

### Scenario E — Dropped Task restored

Expected:

- same Task ID.
- Drop history remains.
- Task returns ACTIVE only with a valid current planned or review date.
- stale past planned date is not silently recommitted.

### Scenario F — Project completes

Expected at model level:

- Project status supports COMPLETED.
- Project-owned Routines are directly queryable.
- 019B performs atomic shutdown behavior.

### Scenario G — Routine resumes after Stop

Expected:

- stopped Routine remains unchanged.
- new Routine gets a new ID.
- `continuationOfRoutineId` links history.

### Scenario H — Routine edited after historical occurrences

Expected:

- old occurrences retain scheduled local date/timezone and status.
- recurrence change affects future derivation only according to 019B/020 policy.

### Scenario I — Goal Tasks all completed

Expected:

- Goal remains ACTIVE until explicit user achievement or abandonment.
- no Goal progress or achievement field changes automatically.

### Scenario J — AI-assisted entity

Expected:

- canonical row `source = AI_ASSISTED` after approval.
- no provider/model field stored on canonical entity.
- user remains authorizing actor in audit history.

---

## 21. Stabilized Proposals Pending Claude Review

- Direct Goal-owned Tasks and Routines are canonical.
- Task and Routine use nullable exclusive parent foreign keys.
- no persisted canonical Plan exists.
- unapproved PlanningDraft persistence is an operational proposal concern.
- active entities retain accepted temporal visibility.
- Task placement is explicit.
- Drop preserves Task identity and Restore reuses it.
- Routine continuation creates a new identity.
- canonical tables remain provider-neutral.
- Carry count, review due, overdue, severity, and Goal progress remain derived.
- physical deletion is not ordinary lifecycle behavior.
- version/concurrency token is required on mutable canonical entities.

---

## 22. Mind Map Impact — Proposed, Do Not Apply Yet

### Product Model

Update ownership diagram to show:

```txt
Goal
├── Project
├── direct Task
└── direct Routine

Task/Routine
→ Goal or Project or standalone
→ exclusive parent
```

### Data Events

Reserve downstream need for:

- lifecycle transitions
- temporal defaults and changes
- Carry history
- Drop and Restore
- Routine continuation
- ownership changes
- correction history

Exact event taxonomy remains 019C.

### Current Decisions

Proposed:

- canonical tables store current truth.
- events preserve meaningful history.
- provider metadata stays outside domain entities.
- derived execution metrics are not stored as Goal truth.
- Task placement and review provenance are explicit.

### Open Questions

Track:

- nullable parent FKs versus polymorphic ownership
- terminal placement retention
- protection normalization
- Routine same-day multiplicity
- RoutineOccurrence userId denormalization
- PlanningDraft temporary persistence

### Technical Study

Add:

- exclusive nullable-FK constraints
- recurrence persistence and versioning
- timezone-safe occurrence identity
- cross-row lifecycle consistency
- cache-versus-source-of-truth discipline

---

## 23. Affected Formal Documents — Record Only

After closure, update or create:

- canonical relational model specification
- ownership and exclusivity constraint specification
- temporal-field and provenance specification
- Task Drop/Restore persistence specification
- Routine continuation identity specification
- RoutineOccurrence persistence specification
- transaction specification in 019B
- event and observability specification in 019C
- runtime DTO and validation rules in 020
- database and scenario tests in 021
- migration plan in 022

Potential ADRs:

- nullable exclusive parent foreign keys for Task and Routine
- provider-neutral canonical entities
- canonical current state plus immutable decision history
- no persisted Goal progress truth
- Routine continuation as new identity

---

## 24. Structured Review Request for Claude

Review this proposal for semantic correctness, relational integrity, temporal consistency, lifecycle preservation, and MVP proportionality.

For every finding provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected section, entity, or invariant
4. concrete failure scenario
5. why the proposal is contradictory, unsafe, incomplete, or over-engineered
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- direct Goal ownership consistency
- exclusive nullable parent foreign keys
- active temporal-checkpoint constraints
- Task placement and terminal-state behavior
- Drop/Restore identity
- Routine continuation identity
- recurrence timezone fields
- occurrence uniqueness
- derived-versus-persisted boundary
- protection representation
- PlanningDraft persistence boundary
- cross-user consistency
- delete restrictions
- missing indexes or constraints

Do not expand into exact SQL dialect, API endpoint design, provider selection, detailed transaction isolation, event-bus technology, raw prompt retention duration, or UI styling.

This discussion remains open until Claude reviews this revision and Mahdi resolves accepted findings.
