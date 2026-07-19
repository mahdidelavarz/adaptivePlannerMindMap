# Discussion 014A — Temporal Checkpoint PlanningDraft Amendment

## Status

Accepted as a required amendment following Discussion 012A.

This amendment extends [[01-Open-Discussions/014-ai-planning-output-contract]] without reopening its accepted hierarchy, seven-day detailed horizon, approval semantics, low-friction conversation policy, or source-of-truth rules.

Related accepted decisions:

- [[01-Open-Discussions/012a-temporal-checkpoint-amendment]]
- [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]
- [[01-Open-Discussions/014-ai-planning-output-contract]]

---

## 1. Accepted PlanningDraft Field Changes

### GoalFields

```txt
GoalFields
- desiredOutcome
- targetDate?
- reviewDate
- reviewDateSource: USER | SYSTEM_DEFAULT
```

Rules:

- `targetDate` remains optional.
- every proposed active Goal must have `reviewDate` before approval.
- absence of a user-provided review date does not create a clarification question.
- the product assigns a visible editable default.
- the default must not be later than an earlier Goal target date.

Default derivation:

```txt
defaultGoalReviewDate = createdLocalDate + 90 local days
Goal.reviewDate = min(defaultGoalReviewDate, targetDate?)
```

### ProjectFields

```txt
ProjectFields
- completionMeaning?
- targetDate?
- reviewDate?
- reviewDateSource?: USER | SYSTEM_DEFAULT
```

Rules:

```txt
ACTIVE proposed Project requires:
targetDate != null
OR
reviewDate != null
```

When `reviewDate` is absent, the product assigns a visible editable default. If a target date exists sooner, review must occur no later than that target.

```txt
defaultProjectReviewDate = createdLocalDate + 30 local days
Project.reviewDate = min(defaultProjectReviewDate, targetDate?)
```

The Project target date remains optional and does not prove completion.

### TaskFields

```txt
TaskFields
- plannedDate?
- reviewDate?
- reviewDateSource?: USER | SYSTEM_DEFAULT
- placement: SCHEDULED | BACKLOG
- deadline?
```

Rules:

```txt
ACTIVE proposed Task requires:
plannedDate != null
OR
reviewDate != null
```

A Task may be intentionally unscheduled while remaining temporally visible:

```txt
plannedDate = null
placement = BACKLOG
reviewDate exists
→ valid active Task
```

When a Task has no `plannedDate`, the product derives a visible editable review default from the accepted planning or Backlog policy. That review date must not occur after a known earlier deadline.

```txt
Task.reviewDate = min(policyReviewDate, deadline?)
```

Exact Task default intervals remain validation policy, not a new conversational question.

### RoutineFields

No additional `reviewDate` is required merely for temporal visibility when an active valid recurrence can derive the next occurrence.

---

## 2. No Added Clarification Friction

Planning AI must not ask solely to obtain checkpoint dates:

```txt
When should this Goal be reviewed?
When should this Project return?
When should this Backlog Task be reconsidered?
```

Instead:

```txt
missing checkpoint
→ deterministic visible default
→ user may edit during draft review
→ approval remains possible
```

Defaults are product policy. They are not AI guesses about user capacity, motivation, lifestyle, or future availability.

---

## 3. Source of Truth and Derived Values

Authoritative fields remain:

```txt
Task execution placement → TaskFields.plannedDate
Task review checkpoint → TaskFields.reviewDate
Project target → ProjectFields.targetDate
Project review checkpoint → ProjectFields.reviewDate
Goal target → GoalFields.targetDate
Goal review checkpoint → GoalFields.reviewDate
Routine execution timing → recurrence
```

`nextTemporalCheckpoint` is derived and must not appear as another independently editable draft field.

---

## 4. Validation Changes

A PlanningDraft is invalid when an included active proposal has no derivable temporal checkpoint.

Additional validation:

- Goal `reviewDate` is required after defaults are applied.
- Project must have `targetDate` or `reviewDate`.
- Task must have `plannedDate` or `reviewDate`.
- Routine must have valid recurrence capable of producing a next occurrence.
- a system-generated review date may not be later than an earlier target date or deadline.
- detailed Task `plannedDate` still obeys the seven-day detailed execution horizon.
- `reviewDate` may fall outside the seven-day execution horizon because it is a management checkpoint, not detailed execution placement.
- default provenance must remain visible and editable.

Repair may add a deterministic missing default. It must not invent a user-specific date through language-model inference.

---

## 5. Review UX Semantics

Draft review must distinguish:

```txt
plannedDate / targetDate
→ intended execution or target timing

reviewDate
→ future commitment review checkpoint
```

System defaults must be marked as defaults rather than represented as user statements.

Backlog remains a valid deliberate placement and is not equivalent to forgotten or invalid work.

---

## 6. First-Week Horizon Interaction

The existing seven-day limit remains unchanged:

- detailed Task execution dates may cover no more than seven consecutive local calendar days.
- Routine first-week projections remain limited to the first execution window.
- Goal, Project, and Task review dates may exist beyond the detailed horizon.
- a review date does not create a Today execution entry.

---

## 7. Affected Later Specifications

Discussion 015 must define execution semantics for `reviewDate`, Backlog placement, and `REVIEW_DUE`.

Discussion 016 must define Reconcile eligibility and presentation without treating review checkpoints as execution failure.

Discussion 017 must consume these canonical fields for bounded continuation and review actions.

Discussion 019 must define persistence and provenance.

Discussion 020 must implement deterministic default assignment before model generation or approval validation.

Discussion 021 must validate timezone boundaries, earlier target/deadline capping, and zero-added-question behavior.

---

## 8. Final Amendment Decisions

- Temporal defaults do not add mandatory planning questions.
- Goal review date is conceptually required and defaulted when absent.
- Project has a target or review checkpoint.
- Task has an execution or review checkpoint.
- Backlog Tasks remain valid and retain review checkpoints.
- Routine recurrence provides its normal temporal checkpoint.
- defaults are visible, editable, and source-labelled.
- `nextTemporalCheckpoint` remains derived.
- review checkpoints do not alter the accepted seven-day detailed execution horizon.
