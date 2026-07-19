# Discussion 012A — Temporal Checkpoint Amendment

## Status

Open for GPT × Claude review.

This amendment formally reopens one narrow part of Discussion 012 without reopening its accepted entity definitions, ownership model, lifecycle meanings, or Goal outcome boundary.

Related accepted or active discussions:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]
- [[01-Open-Discussions/014-ai-planning-output-contract]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/016-reconcile-trigger-and-severity]]
- [[01-Open-Discussions/017-ai-reconcile-intelligence-and-actions]]

---

## 1. Reason for Amendment

The accepted model allows active Tasks, Projects, and Goals without an execution or review date.

This creates a product risk:

```txt
active entity
+ no future execution placement
+ no future review checkpoint
→ temporally invisible commitment
```

Over time, such entities may become:

- forgotten
- difficult to surface honestly
- accumulated in an undated backlog
- impossible to distinguish as intentionally deferred versus accidentally abandoned
- unavailable to deterministic Reconcile timing

The amendment evaluates one principle:

```txt
No ACTIVE entity should become temporally invisible.
```

This is not yet an accepted core invariant. It remains open for review.

---

## 2. Proposed Core Principle

Every active canonical entity should have a derivable future temporal checkpoint.

A temporal checkpoint is not always an execution deadline.

It may mean:

- execute this item on a date
- review whether this item remains active
- reach a target date
- generate the next Routine occurrence

Proposed conceptual mapping:

```txt
Goal
→ reviewDate or targetDate

Project
→ reviewDate or targetDate

Task
→ reviewDate or plannedDate

Routine
→ next occurrence derived from recurrence
```

---

## 3. Proposed Entity Rules

### 3.1 Goal

Proposed fields:

```txt
targetDate?
reviewDate
```

Current proposal:

- `targetDate` remains optional
- `reviewDate` is conceptually required for an ACTIVE Goal
- the product supplies a system default when the user does not provide one
- creation or approval must not require an extra clarification question only to obtain `reviewDate`
- the user may edit the default later

Initial default proposal for validation:

```txt
Goal.reviewDate = createdLocalDate + 90 days
```

The exact interval is not accepted yet.

### 3.2 Project

Proposed fields:

```txt
targetDate?
reviewDate?
```

Current proposal:

```txt
ACTIVE Project requires:
targetDate != null
OR
reviewDate != null
```

When `targetDate` is absent, the product should provide a system-generated `reviewDate` rather than forcing another planning question.

Initial default proposal for validation:

```txt
Project.reviewDate = createdLocalDate + 30 days
```

The exact interval is not accepted yet.

### 3.3 Task

Proposed fields:

```txt
plannedDate?
reviewDate?
```

Current proposal:

```txt
ACTIVE Task requires:
plannedDate != null
OR
reviewDate != null
```

A Task may remain intentionally unscheduled for execution while still having a future review checkpoint.

Therefore:

```txt
plannedDate = null
+ reviewDate exists
→ valid active Task
```

### 3.4 Routine

Routine does not require a separate review checkpoint merely to avoid temporal invisibility when an active recurrence can deterministically produce a next occurrence.

Proposed rule:

```txt
ACTIVE Routine
→ valid recurrence
→ derivable next occurrence
```

Routine-specific periodic review may be introduced later, but is not required by this amendment.

---

## 4. System Defaults Must Not Add Planning Friction

Discussion 013 and Discussion 014 intentionally limit clarification and approval friction.

Therefore the amendment must not introduce mandatory questions such as:

```txt
When should we review this Goal again?
When should this undated Task return?
When should this Project be reconsidered?
```

Default behavior proposal:

```txt
missing user-provided checkpoint
→ product applies visible editable default
→ approval remains possible
```

The default must be visible in review UX but must not restart an interview.

---

## 5. Backlog Remains a Valid Placement

This amendment does not remove Backlog.

Backlog represents an explicit placement decision:

```txt
not scheduled for execution now
```

A review checkpoint represents:

```txt
when the placement or commitment should be reconsidered
```

Therefore:

```txt
Backlog
≠ forgotten forever
```

Current proposal:

- Backlog remains a valid explicit Task placement
- Backlog Tasks still receive a `reviewDate`
- `reviewDate` complements Backlog rather than replacing it

---

## 6. Derived `nextTemporalCheckpoint`

`nextTemporalCheckpoint` must not become an independent canonical source-of-truth field.

It should be derived from authoritative fields.

Conceptual derivation:

```txt
Task.nextTemporalCheckpoint
= earliest existing value among plannedDate and reviewDate

Project.nextTemporalCheckpoint
= earliest existing value among targetDate and reviewDate

Goal.nextTemporalCheckpoint
= earliest existing value among targetDate and reviewDate

Routine.nextTemporalCheckpoint
= next occurrence derived from recurrence
```

Reasons:

- avoids duplicated date truth
- prevents synchronization defects
- preserves the source-of-truth principles accepted in Discussion 014
- supports deterministic queries without adding another editable date

Exact storage or indexing belongs to Discussion 019.

---

## 7. Review Due Is Not Execution Overdue

Two conditions must remain separate:

```txt
EXECUTION_OVERDUE
Task.plannedDate < currentLocalDate
AND Task remains ACTIVE
```

```txt
REVIEW_DUE
entity.reviewDate <= currentLocalDate
AND entity remains ACTIVE
```

`REVIEW_DUE` does not mean execution failed or work is late.

It means an explicit or default management checkpoint has arrived.

Proposed severity guardrail:

```txt
multiple REVIEW_DUE items
+ no overdue execution
≠ automatic MEDIUM or RECOVERY
```

Review checkpoints should be grouped separately from overdue execution and must not inflate Reconcile severity as if each were delayed work.

Discussion 016 requires a later amendment after this model is accepted.

---

## 8. Interaction with Goal Continuation Check

When a Goal review checkpoint arrives, Reconcile may present a deterministic continuation decision.

Proposed options:

```txt
CONTINUE
REVIEW_LATER
ABANDON_GOAL
```

The question must be neutral and fixed:

> Do you still want to continue this Goal?

This mechanism:

- does not ask why execution changed
- does not ask about motivation or emotion
- does not ask the user to predict future capacity
- does not infer intent from absence
- records explicit current intent

Exact Reconcile behavior is defined in the companion Discussion 017 amendment.

---

## 9. Open Questions for Claude

### Q1. Is the principle coherent?

```txt
Every ACTIVE entity must have a derivable future temporal checkpoint.
```

Does this prevent invisible commitments without over-constraining valid long-term work?

### Q2. Are system-generated defaults the correct low-friction mechanism?

Current proposal:

- Goal review default: 90 days
- Project review default when no target date: 30 days
- Task review default when no planned date: derived from planning or Backlog policy

Review whether these defaults should be product rules, configurable policy, or deferred to validation.

### Q3. Should Goal always have `reviewDate`, even when `targetDate` exists?

Current proposal: yes, because target and review answer different questions.

### Q4. Should Project require `reviewDate` only when `targetDate` is absent?

Current proposal: yes for MVP simplicity.

### Q5. Should a Backlog Task always retain a review checkpoint?

Current proposal: yes.

### Q6. Are review-due items sufficiently separated from execution-overdue items?

Review whether any review-due count should affect Reconcile severity, and if so, only through a separately bounded rule.

### Q7. Should the amendment affect standalone entities exactly like owned entities?

Current proposal: yes. Temporal visibility does not depend on ownership.

---

## 10. Required Follow-Up if Accepted

Acceptance requires explicit amendments to:

### Discussion 014

- add proposed Goal, Project, and Task checkpoint fields to `PlanningDraft`
- define visible system defaults
- prevent defaults from creating mandatory clarification
- validate that active proposed entities have a checkpoint

### Discussion 015

- distinguish `plannedDate` from `reviewDate`
- distinguish execution overdue from review due
- define Task placement effects
- preserve Today as `plannedDate = current local date`

### Discussion 016

- add review-checkpoint eligibility semantics
- prevent review-due counts from inflating overdue severity
- define presentation grouping for review checkpoints

### Discussion 017

- define Goal Continuation Check
- define fixed neutral wording and cadence
- keep causal, emotional, motivational, and capacity questions forbidden

### Discussion 019

- persistence, indexes, event history, default provenance, and derived checkpoint queries

### Discussion 020

- deterministic default assignment and rule evaluation

### Discussion 021

- test default generation, timezone boundaries, review-versus-overdue separation, and repeated-prompt suppression

---

## 11. Mind Map Impact — Record Only, Do Not Apply Yet

### Product Vision

Add:

- active commitments must not become temporally invisible
- Backlog is deliberate deferral, not permanent disappearance
- review checkpoints must not be framed as execution failure

### MVP Core Loop

```txt
Create or approve active entity
→ use explicit temporal field when provided
→ otherwise assign visible system review default
→ derive nextTemporalCheckpoint
→ surface execution or review at the correct boundary
```

### User Flow

```txt
undated Task
→ visible review default
→ owner or Backlog placement remains visible
→ review checkpoint arrives
→ keep, schedule, backlog again, or drop
```

### AI Responsibilities

- AI does not invent checkpoint dates from user psychology
- system defaults are deterministic product policy
- AI may explain a visible default but does not need to ask for one

### AI Guardrails

- no active entity with no derivable checkpoint
- no mandatory clarification solely to obtain a review date
- no conflation of review due with execution overdue
- no hidden duplicate `nextTemporalCheckpoint` source of truth

### Data Events

Candidates:

```txt
TEMPORAL_CHECKPOINT_DEFAULTED
TEMPORAL_CHECKPOINT_CHANGED
TEMPORAL_CHECKPOINT_REACHED
ENTITY_REVIEW_DEFERRED
ENTITY_INTENT_REAFFIRMED
```

### Current Decisions

Keep open until review:

- active entities require derivable temporal visibility
- system defaults preserve low-friction planning
- nextTemporalCheckpoint is derived
- Backlog remains explicit placement
- review due remains distinct from execution overdue

---

## 12. Closure Condition

This amendment remains open until:

1. Claude reviews the core invariant and defaults.
2. Mahdi resolves the open questions.
3. Discussions 014–017 receive explicit amendments.
4. Mind Map Impact and Affected Formal Documents are finalized.
