# Discussion 019B — Transactions, Concurrency, and Idempotency

## Status

Open for GPT × Claude review.

This document is Part B of the Discussion 019 persistence program.

Companion discussions:

- [[01-Open-Discussions/019-data-model-and-ai-observability]]
- [[01-Open-Discussions/019a-final-canonical-data-model-resolution]]
- [[01-Open-Discussions/019c-events-ai-observability-and-retention]]

Accepted dependencies:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/015a-temporal-checkpoint-execution-amendment]]
- [[01-Open-Discussions/015b-routine-local-date-and-daily-occurrence-amendment]]
- [[01-Open-Discussions/017b-final-reconcile-intelligence-resolution]]
- [[01-Open-Discussions/018a-final-action-permission-trust-and-reversibility-resolution]]
- [[01-Open-Discussions/018c-final-failure-privacy-domain-and-hostile-input-resolution]]
- [[01-Open-Discussions/019a-final-canonical-data-model-resolution]]

---

## 1. Scope

This discussion defines how accepted canonical mutations execute safely when requests overlap, retry, arrive stale, or affect multiple rows.

It covers:

- transaction boundaries
- optimistic concurrency
- revalidate-before-commit
- stale confirmation rejection
- Project completion and stop cascades
- parent terminal child resolution
- Routine Stop and continuation creation
- Task Drop and Restore
- RoutineOccurrence creation and resolution
- already-materialized future occurrences
- idempotency keys
- retry safety
- duplicate cascade prevention
- reliable event publication boundary
- conflict behavior visible to users

---

## 2. Explicitly Out of Scope

This discussion does not finalize:

- physical SQL syntax for one database
- endpoint routing or DTO shape
- final event taxonomy and event retention
- provider/runtime choice
- queue technology
- final outbox implementation library
- UI visual styling
- offline-first synchronization
- multi-region active-active writes

Those belong to Discussions 019C, 020, and 022.

---

## 3. Governing Mutation Principle

Every consequential write follows:

```txt
receive command
→ identify authenticated user/workspace
→ load current canonical rows
→ validate authorization and invariants
→ compare expected versions
→ revalidate proposal consequences
→ execute all required writes atomically
→ record durable event/outbox facts
→ commit
→ return current authoritative result
```

Generated text is never the transaction authority.

A mutation must either complete coherently or leave canonical state unchanged.

---

## 4. Concurrency Strategy

### 4.1 Optimistic Concurrency by Default

Every mutable canonical entity carries `version` or a database-native concurrency token.

Mutation commands include expected versions for every directly edited entity.

```txt
expectedVersion = currentVersion
→ mutation may proceed

expectedVersion != currentVersion
→ reject as stale
```

The server must not silently overwrite newer state.

### 4.2 When Row Locks Are Required

Optimistic version checks are the default.

Short transaction-scoped row locks or equivalent serialization may be used when:

- a parent transition must read and mutate a child set atomically
- occurrence uniqueness is being established concurrently
- continuation uniqueness is being established
- idempotency ownership is being claimed
- a deterministic cascade must prevent child insertion during the transition

Locks must be scoped to the smallest coherent aggregate and held only inside the transaction.

### 4.3 Conflict Response

A stale command returns a conflict result containing enough deterministic context to refresh:

```txt
CONFLICT_STALE_VERSION
- entityId
- expectedVersion
- currentVersion
- currentStatus
- changedMaterialFields where safe
```

The user-facing flow must state that nothing was changed and regenerate any affected preview.

No automatic retry may reinterpret intent after a conflict.

---

## 5. Revalidate Before Commit

Discussion 018A requires independent commit-time validation.

Immediately before mutation, the product reloads or locks current state and verifies:

- lifecycle status
- ownership and authorization
- entity version
- protection status
- deadline context
- temporal placement
- selected entity set
- parent/child relationships
- rule eligibility where applicable
- consequence preview assumptions

If any material assumption changed:

```txt
no mutation
→ confirmation expires
→ refreshed preview required
→ explicit confirmation required again
```

A previous invalidation event is not sufficient proof that the proposal remains valid.

---

## 6. Command and Idempotency Model

Every consequential mutation command carries an idempotency key scoped to:

```txt
user/workspace
command type
client request or confirmation identity
```

Proposed idempotency record:

```txt
IdempotencyRecord
- id
- userId
- idempotencyKey
- commandType
- requestHash
- status: IN_PROGRESS | SUCCEEDED | FAILED_RETRYABLE | FAILED_FINAL
- resultReference?
- createdAt
- completedAt?
- expiresAt
```

Uniqueness:

```txt
UNIQUE(userId, idempotencyKey)
```

Rules:

- same key and same request hash returns the existing result
- same key and different request hash is rejected
- duplicate requests do not repeat canonical mutations
- an expired record does not permit repeating a command whose domain state already proves completion
- idempotency does not bypass version checks or authorization

### 6.1 Failure Semantics

If transaction commit status is uncertain to the caller:

- caller retries with the same idempotency key
- server resolves the existing record and domain result
- caller must not generate a new key merely because the response was lost

`FAILED_RETRYABLE` means the command did not commit canonical effects.

A committed command may never be marked retryable merely because post-commit response delivery failed.

---

## 7. Task Mutation Transactions

### 7.1 Complete Task

Transaction:

```txt
lock/load Task
→ verify ACTIVE and expected version
→ revalidate parent and protection rules
→ set status COMPLETED
→ set terminalAt
→ advance version
→ write audit/outbox event
→ commit
```

Completion does not change parent Project or Goal status automatically.

### 7.2 Drop Task

Transaction:

- verify Task is ACTIVE
- verify current protection and deadline rules
- verify selected bulk set when applicable
- preserve placement and historical dates
- set status DROPPED and `terminalAt`
- record parent-empty consequence fact when applicable
- do not terminate parent automatically

### 7.3 Restore Task

Transaction:

- verify Task is DROPPED
- compare expected version
- validate current parent remains active or resolve structural conflict first
- require a valid current plannedDate or reviewDate
- never silently reuse an expired past plannedDate
- set status ACTIVE
- clear `terminalAt`
- update accepted placement/checkpoint
- record Restore event

Restore uses the same Task identity.

### 7.4 Concurrent Task Scenarios

```txt
Complete vs Drop
→ first committed version wins
→ second command receives stale conflict

Restore vs parent termination
→ parent terminal transaction wins if locked first
→ Restore must revalidate parent and fail or require reparenting

Carry/replan vs Complete
→ no lost update; version conflict forces refresh
```

---

## 8. RoutineOccurrence Transactions

### 8.1 Occurrence Creation

Occurrence creation uses:

```txt
UNIQUE(routineId, scheduledLocalDate)
```

Creation algorithm:

- load Routine and recurrence version
- verify scheduled date is inside the inclusive effective range
- verify Routine eligibility for that date
- insert occurrence
- if uniqueness conflict occurs, return the existing occurrence

A retry never creates a second occurrence.

### 8.2 Occurrence Resolution

Transaction:

- load occurrence and parent Routine ownership
- verify expected occurrence version
- permit `PENDING → DONE | MISSED`
- set `resolvedAt`
- advance version
- record event

Corrections to historical occurrences are explicit audited transitions defined by Discussion 015; they are not silent overwrites.

### 8.3 Race with Routine Stop

A resolution of an already-valid occurrence may proceed after Routine Stop when the occurrence date was eligible and the occurrence already existed.

A new occurrence may not be created outside the final effective date.

The transaction must evaluate the occurrence's scheduled local date against the Routine's accepted effective range, not merely its current ACTIVE/STOPPED status.

---

## 9. Routine Stop Transaction

Manual Stop transaction:

```txt
load/lock Routine
→ verify ACTIVE and expected version
→ determine final effective local date
→ set status STOPPED
→ set stoppedAt
→ set effectiveUntilLocalDate
→ advance version
→ handle future materialized occurrences
→ write events/outbox
→ commit
```

### 9.1 Future Materialized Occurrences

Proposed MVP rule:

- occurrences inside the accepted effective range remain historical/valid
- PENDING occurrences after `effectiveUntilLocalDate` are invalid future materializations
- invalid future occurrences are not marked MISSED
- they are removed only if they have never been user-resolved and no external reference requires preservation
- otherwise they are retained with an explicit invalidated/cancelled event representation defined in 019C

Claude should review whether physical deletion of never-observed future PENDING occurrences is acceptable or whether an explicit CANCELLED/VOID state is required.

### 9.2 Stop vs Occurrence Resolution Race

The Routine row and affected occurrence row are validated inside one coherent locking/version strategy.

If occurrence resolution commits first, Stop preserves that resolved occurrence.

If Stop changes the effective range first, resolution remains allowed only if the occurrence date stays inside the accepted range.

---

## 10. Routine Continuation Creation

Creating a continuation Routine is one transaction:

- load/lock source Routine
- verify source is STOPPED
- verify same user/workspace
- verify source has no existing direct continuation
- validate no self-reference or cycle
- create new Routine with new ID
- set `continuationOfRoutineId`
- set explicit effectiveFromLocalDate
- preserve source Routine unchanged
- write lineage event

Uniqueness on `continuationOfRoutineId` prevents branching in MVP.

Concurrent continuation attempts:

```txt
first commit succeeds
second receives uniqueness/conflict result
```

The product returns the existing continuation when the repeated request uses the same idempotency key.

---

## 11. Project Completion Transaction

Project completion is a single atomic transaction.

Preconditions:

- Project is ACTIVE
- expected Project version matches
- active child Task resolution requirements have been satisfied
- current child relationships are loaded inside the transaction
- confirmation preview remains current

Mutation order:

```txt
1. lock Project
2. lock/query current Project-owned active Routines
3. revalidate active child Task and structural state
4. transition Project ACTIVE → COMPLETED
5. set Project terminalAt
6. transition every active Project-owned Routine → STOPPED
7. set each Routine stoppedAt and effectiveUntilLocalDate
8. handle invalid future occurrences under the accepted policy
9. advance all affected versions
10. write domain/audit/outbox records
11. commit
```

All or none of the Project and Routine changes commit.

Goal-owned and standalone Routines are unaffected.

Parent Goal status remains unchanged.

### 11.1 Child Insert Race

A new Task or Routine may not be attached to a Project while its terminal transition is committing.

Attachment must validate and lock/read the current Project status immediately before commit.

If the Project is no longer ACTIVE, attachment fails without mutation.

---

## 12. Project Stop Transaction

Project Stop uses the same atomic child-Routine shutdown invariant as completion unless a later accepted product decision distinguishes it.

```txt
Project ACTIVE → STOPPED
→ all active Project-owned Routines STOPPED
```

Active child Task resolution follows the accepted parent lifecycle contract from Discussion 015.

No child remains illegally active under a terminal Project after commit.

The transaction distinguishes Project status outcome from Goal outcome; neither COMPLETED nor STOPPED changes Goal status automatically.

---

## 13. Goal Terminal Transactions

Goal ACHIEVED or ABANDONED requires:

- explicit user confirmation
- current Goal version
- current child set
- accepted resolution for active Projects, direct Tasks, and direct Routines
- atomic application of all chosen child resolutions where the flow is represented as one confirmed operation

Two acceptable command shapes are proposed:

### Option A — Single Aggregate Command

One confirmation contains every child resolution and commits all changes atomically.

### Option B — Resolution Staging then Final Goal Transition

Child resolutions commit separately; final Goal transition revalidates that no illegal active children remain.

Current proposal for MVP:

```txt
Option B
```

Reason:

- smaller transaction scope
- clearer conflict handling
- easier recovery
- user can resolve complex children incrementally

The final Goal transition remains blocked until structural invariants are satisfied.

Claude should review whether Option B weakens the user's expectation of one confirmed parent action.

---

## 14. Bulk Action Transactions

Bulk actions use one command and one transaction only when:

- selected IDs are explicit
- every item supports the same action
- all expected versions are supplied
- shared eligibility remains valid
- all consequences were previewed

Default atomicity:

```txt
all selected items succeed
OR
none mutate
```

Partial bulk success is rejected for MVP because it makes confirmation meaning ambiguous.

If one selected item became stale, the entire bulk command fails and the selection preview is regenerated.

---

## 15. Reliable Event Publication

Canonical mutation and durable event intent must share one transaction.

Proposed model:

```txt
domain rows
+ audit/event records or transactional outbox rows
→ same database transaction
```

Post-commit publishers may retry delivery idempotently.

The system must not:

- commit domain state and lose the corresponding durable event intent
- publish an event before the transaction commits
- treat message-broker delivery as the source of canonical truth

Exact event envelope and retention belong to Discussion 019C.

---

## 16. Deadlock and Ordering Discipline

When multiple rows are locked, the application uses deterministic ordering.

Proposed order:

```txt
parent before child
entity type order: Goal → Project → Task → Routine → RoutineOccurrence
within type: ascending stable ID
idempotency record before executing domain mutation
```

Retry on database deadlock is permitted only with the same command and idempotency key.

A deadlock retry must re-run all authorization, version, and consequence checks.

---

## 17. Failure Matrix

```txt
Condition                         Canonical Result
--------------------------------------------------------------------
Stale entity version              no mutation; conflict
Duplicate idempotency key         return prior result or reject hash mismatch
Duplicate occurrence insert       return existing occurrence
Concurrent Task terminal actions  first commit wins; second conflicts
Parent terminal during attach     attach rejected
Duplicate Routine continuation    first commit wins; second conflicts
Cascade child mutation failure    full transaction rollback
Outbox publisher failure          domain commit remains; delivery retries
Lost HTTP response after commit   same-key retry returns committed result
Partial bulk staleness             entire bulk transaction rejected
```

---

## 18. Initial Transaction Contracts

```txt
Command                         Atomic Boundary
--------------------------------------------------------------------
COMPLETE_TASK                   Task + audit/outbox
DROP_TASK                       Task + parent-empty fact + audit/outbox
RESTORE_TASK                    Task + placement + audit/outbox
RESOLVE_OCCURRENCE              Occurrence + audit/outbox
STOP_ROUTINE                    Routine + future occurrence handling + events
CREATE_ROUTINE_CONTINUATION     source validation + new Routine + lineage event
COMPLETE_PROJECT                Project + active child Routine shutdown + events
STOP_PROJECT                    Project + active child Routine shutdown + events
BULK_TASK_ACTION                all selected Tasks + events
FINALIZE_GOAL_TERMINAL          Goal after child invariants + events
```

---

## 19. Open Questions for Claude

### Q1. Is optimistic concurrency plus short scoped locks the right default?

Review whether any aggregate requires serializable isolation rather than explicit row/version discipline.

### Q2. Is the idempotency record model sufficient?

Review request-hash mismatch, uncertain commit, expiration, and domain-result recovery.

### Q3. Should bulk actions be fully atomic in MVP?

Current proposal: yes. Any stale or invalid item rejects the entire selected set.

### Q4. Is the Project terminal cascade ordering coherent?

Review child insertion races, Routine edits, occurrence resolution, and Project version changes.

### Q5. How should invalid future PENDING RoutineOccurrences be represented?

Options:

```txt
A. physically delete never-resolved future rows
B. retain rows with CANCELLED/VOID status
C. retain separate invalidation records without adding occurrence status
```

Current proposal: A only when the row has never been exposed/resolved and has no external reference; otherwise C.

### Q6. Should occurrence resolution after Routine Stop remain allowed?

Current proposal: yes, when the occurrence's local date remains inside the final effective range.

### Q7. Is single-continuation enforcement correct for MVP?

Review whether `UNIQUE(continuationOfRoutineId)` is sufficient and whether cycle detection requires lineage traversal or a root identifier.

### Q8. Should Goal terminal transitions use staged child resolution?

Current proposal: resolve children through smaller transactions, then revalidate and commit the Goal transition.

### Q9. Is all-or-nothing bulk mutation too strict?

Review usability versus confirmation integrity.

### Q10. Is transactional outbox required for MVP?

Current proposal: yes for durable audit/event intent, even if delivery is initially simple.

### Q11. What lock ordering gaps could still deadlock?

Focus on simultaneous Project terminal cascade, Routine Stop, occurrence resolution, and child attachment.

### Q12. Does duplicated `userId` on RoutineOccurrence need database-level composite foreign-key enforcement?

Current proposal: yes where supported; otherwise enforce transactionally and test aggressively.

### Q13. What should happen when idempotency records expire?

Review safe replay detection for lifecycle commands whose domain result may remain observable indefinitely.

### Q14. Are conflict responses revealing too much or too little?

Review privacy-safe material-field summaries and multi-device refresh needs.

### Q15. Are any accepted Discussion 018A confirmation guarantees lost in this transaction model?

Focus on current selection set, consequences, protection, deadlines, and revalidation.

---

## 20. Scenario Checks

### Scenario A — Two devices complete the same Task

Expected:

- both submit the same expected version
- one commits
- the other receives stale conflict
- no second event or terminal transition is created

### Scenario B — Response is lost after Task creation commits

Expected:

- client retries with same idempotency key
- server returns original created entity
- no duplicate entity is created

### Scenario C — Project completion races with Routine edit

Expected:

- deterministic lock/version order chooses one commit
- the losing operation refreshes
- no active Project-owned Routine remains under a completed Project

### Scenario D — Occurrence resolution races with Routine Stop

Expected:

- resolved occurrence is preserved when its date remains eligible
- no invalid future occurrence becomes MISSED
- transaction outcome is auditable

### Scenario E — Two continuation requests race

Expected:

- one continuation Routine is created
- second request returns conflict or same idempotent result
- no branching lineage

### Scenario F — One Task in bulk Drop becomes protected

Expected:

- entire bulk command fails
- nothing is dropped
- preview is regenerated with the protected Task excluded or explained

### Scenario G — Child Routine is attached during Project completion

Expected:

- attachment or completion waits/conflicts according to lock order
- no child commits beneath a terminal Project

### Scenario H — Outbox publisher is unavailable

Expected:

- canonical transaction and outbox row commit
- user receives canonical success
- publisher retries later
- event delivery is not duplicated semantically

---

## 21. Stabilized Proposals Pending Claude Review

- optimistic concurrency is the default.
- scoped locks protect multi-row invariant transitions.
- every confirmed mutation is revalidated before commit.
- stale writes never silently overwrite current state.
- consequential commands are idempotent.
- Project terminal transitions and child Routine shutdown are atomic.
- duplicate occurrence creation resolves to the existing daily occurrence.
- Task terminal races use first-commit-wins through version checks.
- bulk mutations are all-or-nothing in MVP.
- domain mutation and durable event intent share one transaction.
- Goal child resolution is staged before final Goal terminal commit.

---

## 22. Mind Map Impact — Proposed, Do Not Apply Yet

### Product Vision

- user-confirmed intent is protected against retries and concurrent edits.
- failure or duplicated submission never creates duplicated consequences.
- history and current state remain transactionally consistent.

### MVP Core Loop

```txt
validated command
→ current-state reload
→ version and permission checks
→ consequence revalidation
→ atomic mutation
→ durable event intent
→ idempotent result
```

### User Flow

Add:

- stale-preview refresh state
- conflict explanation without lost user input
- retry-safe confirmation submission
- all-or-nothing bulk conflict state
- Project cascade progress represented as one committed result

### AI Responsibilities

- AI proposals include explicit entity IDs and expected versions through product context.
- AI never retries mutations by inventing new confirmation identity.
- AI explanation cannot override conflict or transaction outcome.

### AI Guardrails

- no stale confirmation commit
- no duplicate mutation after response loss
- no partial bulk success
- no child under terminal parent
- no duplicate daily occurrence
- no duplicate Routine continuation

### Data Events for Discussion 019C

Candidates:

```txt
COMMAND_RECEIVED
COMMAND_CONFLICTED
COMMAND_IDEMPOTENT_REPLAYED
TRANSACTION_COMMITTED
TRANSACTION_ROLLED_BACK
CASCADE_COMPLETED
BULK_ACTION_REJECTED
OCCURRENCE_DUPLICATE_PREVENTED
OUTBOX_DELIVERY_RETRIED
```

### Traction and Guardrail Metrics

- stale command rate
- duplicate submission replay rate
- mutation conflict rate
- cascade rollback rate
- duplicate occurrence prevention rate
- idempotency hash mismatch rate
- partial bulk mutation rate, expected zero
- child-under-terminal-parent rate, expected zero
- duplicate continuation rate, expected zero

### Current Decisions

Proposed:

- versioned optimistic concurrency
- scoped locking for cross-row invariants
- all-or-nothing bulk mutation
- transactional durable event intent
- first-commit-wins for incompatible terminal transitions

### Open Questions

- future occurrence invalidation representation
- Goal staged resolution versus one aggregate transaction
- idempotency expiration and replay detection
- exact composite ownership enforcement
- minimum lock/isolation strategy

---

## 23. Affected Formal Documents — Record Only

After closure, update or create:

- transaction-boundary specification
- optimistic-concurrency contract
- stale-confirmation implementation contract
- idempotency-key and replay specification
- Project terminal cascade specification
- Routine Stop and future-occurrence policy
- bulk mutation atomicity specification
- transactional outbox ADR
- concurrency and race test plan in Discussion 021
- rollout and migration sequencing in Discussion 022

---

## 24. Structured Review Request for Claude

Review this exact proposal for correctness, race safety, transactional coherence, and proportional MVP complexity.

For every finding, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected section or transaction
4. concrete concurrent or retry failure scenario
5. why the proposal is unsafe, contradictory, or incomplete
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- lock ordering and deadlocks
- stale confirmation enforcement
- Project cascade atomicity
- occurrence Stop/resolution races
- invalid future occurrence handling
- idempotency after uncertain commit
- duplicate Routine continuation
- bulk action atomicity
- Goal terminal staging
- reliable event publication
- cross-user ownership enforcement

Do not expand into endpoint naming, specific ORM APIs, UI styling, provider selection, final event retention, or multi-region architecture.

This discussion remains open until Claude reviews the proposal and Mahdi resolves accepted findings.