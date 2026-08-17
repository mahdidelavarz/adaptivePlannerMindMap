# Discussion 026 — Backlog Removal, Parent-Owned Undated Tasks, and Quick Capture

## Status

**OPEN — proposed direction, pending Claude review.**

This discussion is **not yet accepted or closed**.

Nothing in this file is authoritative product behavior until review is completed and the resulting direction is explicitly accepted.

```txt
STATUS = OPEN
CLAUDE_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
FORMAL_DOC_RECONCILIATION = PENDING
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

Do not update `00-Canvas/Planner-Mindmap.canvas`, formal specifications, implementation plans, runtime contracts, or prototype behavior from this file while it remains open.

This discussion reopens a broad Task temporal-model decision family. It is not a UI-only cleanup.

Primary affected accepted discussions include:

- [[01-Closed-Discussions/012-core-product-model]]
- [[01-Closed-Discussions/012a-temporal-checkpoint-amendment]]
- [[01-Closed-Discussions/014-ai-planning-output-contract]]
- [[01-Closed-Discussions/014a-temporal-checkpoint-planning-draft-amendment]]
- [[01-Closed-Discussions/015-task-and-routine-execution-model]]
- [[01-Closed-Discussions/015a-temporal-checkpoint-execution-amendment]]
- [[01-Closed-Discussions/016-reconcile-trigger-and-severity]]
- [[01-Closed-Discussions/016a-review-checkpoint-trigger-and-presentation-amendment]]
- [[01-Closed-Discussions/017-ai-reconcile-intelligence-and-actions]]
- [[01-Closed-Discussions/019a-canonical-data-model-and-invariants]]
- [[01-Closed-Discussions/019c-events-ai-observability-and-retention]]
- [[01-Closed-Discussions/020b-api-and-frontend-state-contracts]]
- [[01-Closed-Discussions/020c-structured-output-reliability-and-cost-controls]]
- [[01-Closed-Discussions/022-updated-mvp-implementation-plan]]
- [[01-Closed-Discussions/023-persistent-planning-facts-and-rolling-execution-context]]
- [[01-Closed-Discussions/025-task-dependency-sequences-and-hierarchical-reconcile-grouping]]

---

## 1. Problem Statement

The accepted Task model currently supports explicit Backlog placement:

```txt
Task.placement = SCHEDULED | BACKLOG
```

Backlog Task semantics currently include:

```txt
plannedDate = null
reviewDate exists
placement = BACKLOG
```

This was introduced to keep intentionally unscheduled Tasks temporally visible.

The resulting model, however, creates unnecessary user-facing and domain complexity around concepts such as:

```txt
Backlog
Review Due
Carry
Scheduled
Keep in Backlog
Move to Backlog
```

For a simple Task, this asks the user and the product to maintain distinctions that are not all necessary to preserve the actual commitment.

A second issue exists for Tasks that already belong to a Goal or Project. Such Tasks can remain meaningful without an individual execution date because they are part of a larger active commitment.

Example:

```txt
Project: Launch Website

Tasks:
- Design UI          Aug 20
- Implement Auth     Aug 24
- Write Docs         no plannedDate
```

`Write Docs` is not forgotten merely because it has no own date. It remains part of the Project commitment and can be surfaced through the Project's review/lifecycle context.

A third issue exists in Quick Capture. Users need to record something immediately without deciding whether it is a Task, Routine, Goal-owned Task, Project-owned Task, or standalone dated Task at capture time.

The current Task + Backlog model tries to force this unresolved input into canonical Task semantics too early.

---

## 2. Core Product Direction

The proposed governing direction is:

```txt
Capture ≠ Commitment.
```

```txt
Standalone Task
→ owns its execution date.
```

```txt
Parent-owned Task
→ may have an explicit execution date
OR
→ may delegate temporal resurfacing to its direct parent.
```

```txt
Backlog
→ removed completely from MVP.
```

The goal is to preserve this invariant:

> No active commitment should become unreachable or forgotten.

But that invariant no longer means:

> Every active Task must own an independent temporal checkpoint.

---

## 3. Backlog Is Removed Completely

This is not only a UI removal.

If this discussion is accepted, the following concepts must be removed or explicitly replaced across product, persistence, events, AI, API, Reconcile, and implementation specifications:

```txt
Task.placement = SCHEDULED | BACKLOG
BACKLOG placement
PLACEMENT_CHANGED_TO_BACKLOG
PLACEMENT_CHANGED_TO_SCHEDULED where it exists only as Backlog counterpart
KEEP_IN_BACKLOG_WITH_NEW_REVIEW_DATE
IN_BACKLOG_UNSCHEDULED
Task-specific Backlog review semantics
Task Backlog filters/routes if any remain
```

If `Task.placement` has no remaining independent meaning after Backlog removal, the field should be removed entirely rather than retained as redundant state.

### Archive is not Backlog

Archive remains a separate possible visibility/history concern.

```txt
Archive ≠ Backlog
Archive ≠ lifecycle state
```

For example:

```txt
COMPLETED Project
→ may later be hidden from ordinary active views through Archive behavior
```

Archive does not preserve an unfinished commitment and is not a deferral mechanism.

### Freeze / Pause is not Backlog

A future Phase-2 concept may support:

```txt
Freeze / Pause
→ Continue / Resume
```

meaning:

> this commitment is still valid, but active work is intentionally suspended.

This is explicitly out of MVP scope and must not be introduced merely to replace Backlog.

---

## 4. Revised Task Temporal Model

### 4.1 Standalone ACTIVE Task

A canonical standalone ACTIVE Task must have an execution date.

```txt
Task.status = ACTIVE
AND goalId = null
AND projectId = null
→ plannedDate REQUIRED
```

If a user input has no date and no parent, it is not yet promoted to a complete standalone canonical Task.

### 4.2 Goal-owned ACTIVE Task

A direct Goal-owned Task may have no individual planned date.

```txt
Task.status = ACTIVE
AND goalId != null
AND projectId = null
→ plannedDate OPTIONAL
```

### 4.3 Project-owned ACTIVE Task

A Project-owned Task may have no individual planned date.

```txt
Task.status = ACTIVE
AND projectId != null
→ plannedDate OPTIONAL
```

Project-owned Task Goal context remains derived through `Project.goalId`; do not duplicate ownership.

### 4.4 No fake inherited execution window

An undated parent-owned Task does not inherit or persist a synthetic execution window such as:

```txt
[createdAt → parent.targetDate]
```

and must not derive an execution start from `createdAt`.

```txt
createdAt ≠ execution startDate
parent.reviewDate ≠ child deadline
parent.targetDate ≠ child plannedDate
```

The Task remains valid because it is inside a parent planning context, not because fake dates are created for it.

---

## 5. Task `reviewDate` Is Proposed for Removal

Under the revised model, no remaining Task category requires a Task-owned `reviewDate` for MVP.

### Standalone Task

```txt
plannedDate required
→ Task.reviewDate not needed
```

### Parent-owned scheduled Task

```txt
plannedDate exists
→ Task.reviewDate not needed for visibility
```

### Parent-owned unscheduled Task

```txt
plannedDate absent
→ temporal responsibility belongs to direct parent
→ Task.reviewDate not needed
```

Therefore the proposed Task shape becomes conceptually:

```txt
Task
- plannedDate?
- deadline?
```

with ownership-sensitive validity rules.

`reviewDate`, `reviewDateSource`, and Task-specific `REVIEW_DUE` semantics should be removed unless review discovers an independent remaining use.

---

## 6. Temporal Visibility Becomes Responsibility-Based

Discussion 012A currently states:

```txt
No ACTIVE entity may become temporally invisible.
```

That wording is intentionally reopened.

The proposed replacement is:

> No ACTIVE commitment may become temporally unreachable.

Temporal responsibility is assigned deterministically:

```txt
Standalone ACTIVE Task
→ own plannedDate

Parent-owned ACTIVE Task with plannedDate
→ own plannedDate

Parent-owned ACTIVE Task without plannedDate
→ direct parent's temporal review/lifecycle checkpoint

ACTIVE Routine
→ recurrence / next occurrence

ACTIVE Goal / Project
→ own target/review policy
```

This is a commitment-tree-level invariant rather than a requirement that every child entity stores its own date.

### Direct ownership rule

Temporal responsibility follows direct ownership.

```txt
Project-owned undated Task
→ surfaced through Project review

Direct Goal-owned undated Task
→ surfaced through Goal review
```

A Goal review should not independently duplicate review of Project-owned Tasks already governed through their Project.

---

## 7. Goal / Project Review Date Is Product Policy, Not Required User Input

A major UX correction is proposed for Goal and Project `reviewDate`.

The product should not require the user to answer questions such as:

```txt
When do you want to review this Project?
When should this Goal be reconsidered?
```

Most users do not have a meaningful answer during creation.

### User-facing rule

`reviewDate` should not be required in the normal Goal/Project creation flow.

If the user explicitly provides a review date through an advanced/edit flow, it may be honored.

Otherwise the product derives the checkpoint deterministically.

### Default when targetDate exists

Proposed rule:

```txt
if targetDate exists
→ reviewDate = targetDate
```

Rationale:

The target itself is the natural boundary at which an ACTIVE unfinished Goal/Project needs review.

Do not default review to a date after target merely for convenience.

### Default when targetDate does not exist

Initial MVP policy:

```txt
Project.reviewDate = createdLocalDate + 30 local days
Goal.reviewDate = createdLocalDate + 90 local days
```

These values are deterministic product policy, not AI guesses.

### After a continuation review

If the entity remains ACTIVE after review:

```txt
Project next default review
→ currentLocalDate + 30 local days

Goal next default review
→ currentLocalDate + 90 local days
```

If an authoritative target exists earlier than the next default checkpoint:

```txt
reviewDate = min(defaultNextReviewDate, targetDate)
```

### UX visibility

`reviewDate` may remain canonical product data while being absent from the normal creation form.

The user should not be forced to manage a system safety-net field merely because the domain model needs a future resurfacing boundary.

---

## 8. Parent Review Must Include Undated Direct Child Tasks

When a Goal or Project enters its accepted review flow, all directly owned ACTIVE child Tasks without `plannedDate` must be included deterministically.

Example:

```txt
Project Review

Completed:
✓ Design UI
✓ Implement Auth

Still Active / unscheduled:
○ Write Docs
○ QA
```

Required rule:

```txt
Parent-owned ACTIVE Task
AND plannedDate = null
→ included in direct Parent review context
```

This is not optional recommendation logic and must not depend on AI judgment.

### Scheduled children

Scheduled ACTIVE child Tasks may also be summarized when relevant, but the undated direct children must not be omitted.

### No duplicate traversal

For:

```txt
Goal
└ Project
   └ Task without plannedDate
```

The Project owns the Task's temporal review responsibility.

Goal review may surface the unresolved Project, but should not duplicate the Task as a separate direct Goal review item.

---

## 9. Today Remains Strict

Today remains execution-only.

```txt
Task appears in Today
iff
Task.status = ACTIVE
AND Task.plannedDate = currentLocalDate
```

An undated Task does not enter Today merely because its Goal or Project is ACTIVE or under review.

No parent target/review date is inherited as a child Today date.

---

## 10. Carry Semantics

Carry remains an execution-date mutation.

```txt
old plannedDate
→ new plannedDate
```

Therefore:

```txt
parent-owned Task with plannedDate = null
→ has nothing to Carry
```

It may instead be:

```txt
scheduled to a date
completed
dropped
edited
```

Completing an undated parent-owned Task must not require first assigning a fake planned date.

---

## 11. Quick Capture UX

Quick Capture is a real product need but should not become a new visible ontology burden.

The user may see the ordinary Add Task interaction:

```txt
Add Task

Title: Call dentist
Date: optional
Parent: optional
```

If the user submits without both a date and a parent, the UI should not block capture or force an immediate decision.

The interaction remains fast.

What changes is the internal state created by that submission.

---

## 12. CaptureItem — Supporting Persisted Record

The proposed internal concept is:

```txt
CaptureItem
```

This name is provisional for review but preferred over `DraftTask` because captured input may later become a Routine rather than a Task.

`CaptureItem` is not a sixth canonical work entity.

It is a durable supporting record for unresolved captured input.

Conceptual minimal shape:

```txt
CaptureItem
- id
- userId
- title
- createdAt
- updatedAt
- source
- version
```

Additional fields should be added only when review finds a concrete need.

It must be persisted so app close/reconnect does not lose the capture.

### Capture is not canonical work truth

Until resolved:

```txt
CaptureItem
≠ canonical Task
≠ Routine
≠ execution commitment
```

---

## 13. Promotion / Resolution Rules

A captured item becomes eligible to create a canonical Task when:

```txt
plannedDate exists
OR
Goal ownership exists
OR
Project ownership exists
```

Examples:

```txt
Call dentist
→ Set date Aug 20
→ create standalone canonical Task
```

```txt
Research WebGPU
→ Attach to Project
→ create Project-owned canonical Task
```

A captured item may instead become a Routine:

```txt
Read every morning
→ Convert to Routine
```

Or it may be discarded.

### Identity rule

Promotion should not mutate the CaptureItem row into a Task or Routine identity.

Preferred conceptual behavior:

```txt
resolve CaptureItem
→ create canonical entity
→ preserve audit linkage
```

because capture identity and canonical work identity have different semantics.

The exact resolved-state / retention model for CaptureItem requires review with Discussion 019C.

---

## 14. Capture ≠ Commitment

This is a core invariant.

Before resolution, a CaptureItem:

- does not enter Today,
- does not become execution-overdue,
- does not have Carry semantics,
- does not count as execution failure,
- does not contribute to repeated-Carry evidence,
- does not contribute to Adaptive Planning failure/capacity evidence,
- does not become a Task merely because time passes.

Age may be shown as neutral capture context if useful, but age does not convert capture into execution debt.

---

## 15. Reconcile and Quick Capture

Unresolved CaptureItems are surfaced deterministically through Reconcile because they need classification or commitment resolution.

Example navigation:

```txt
Reconcile  • 3
```

Possible Reconcile section:

```txt
Needs attention (3)

- Call dentist
- Research WebGPU
- Read every morning
```

Allowed resolution families:

```txt
SET_DATE
ATTACH_TO_GOAL
ATTACH_TO_PROJECT
CONVERT_TO_ROUTINE
DISCARD
```

Exact copy and UI are out of scope until the domain contract is accepted.

### Severity isolation

CaptureItems must not inflate execution severity.

Proposed Reconcile context separation:

```txt
executionSeverity
commitmentReviews
unresolvedCaptureCount
```

Rule:

```txt
unresolved CaptureItem
→ Reconcile-visible
→ contributes to unresolvedCaptureCount
→ does NOT contribute to actionable execution backlog count
→ does NOT contribute to oldest execution-unresolved age
→ does NOT independently produce LIGHT / MEDIUM / RECOVERY escalation
```

A Reconcile badge may include unresolved captures, but badge semantics must remain distinguishable from execution severity.

---

## 16. PlanningDraft Changes

The accepted PlanningDraft Task validation is reopened.

Current rule roughly requires:

```txt
ACTIVE proposed Task
→ plannedDate OR reviewDate
```

Proposed replacement depends on ownership after draft validation:

```txt
if proposed Task is standalone
→ plannedDate required

if proposed Task is direct Goal-owned or Project-owned
→ plannedDate optional
```

Remove Task-specific draft fields if no longer needed:

```txt
placement
reviewDate
reviewDateSource
```

Goal/Project reviewDate defaulting remains deterministic product policy and should not create mandatory AI clarification.

PlanningDraft should not create CaptureItems merely because an AI-generated Task lacks enough temporal/ownership data; invalid AI work proposals should follow accepted clarification/validation rules rather than silently entering Quick Capture.

Quick Capture is a user capture flow, not a repair bucket for weak AI output.

---

## 17. Parent Terminal Transitions

An undated Task remains an ACTIVE child Task for lifecycle purposes.

Therefore parent terminal rules do not exempt it.

Example:

```txt
Complete Project

ACTIVE children:
- QA — planned
- Write Docs — no plannedDate
```

Both must participate in the accepted child-resolution flow.

```txt
plannedDate = null
≠ resolved
≠ ignorable
```

Existing Project/Goal terminal semantics should be amended only where they incorrectly depend on Task placement or Task reviewDate.

---

## 18. Discussion 025 — Dependency Sequence Amendment

Discussion 025 currently contains accepted Backlog-specific sequence rules.

Those rules are explicitly reopened if Discussion 026 is accepted.

Replace:

```txt
ACTIVE predecessor in BACKLOG
→ unresolved
→ downstream BLOCKED
```

with:

```txt
ACTIVE parent-owned predecessor
AND plannedDate = null
→ unresolved
→ downstream BLOCKED
```

A sequence member without `plannedDate` can exist only when canonical ownership makes that Task valid.

Standalone sequence members still require `plannedDate` while ACTIVE.

### CARRY_ALL preview

Replace Backlog classification:

```txt
IN_BACKLOG_UNSCHEDULED
```

with a concept such as:

```txt
UNSCHEDULED_PARENT_OWNED
```

A currently undated parent-owned sequence member must not be silently assigned a date by `CARRY_ALL`.

Preview must explicitly disclose and require confirmation if the grouped replan will schedule that Task.

All other accepted 025 safeguards remain unless review finds a contradiction:

- stable `sequenceId`,
- hard dependency,
- deadline validation,
- protected manual scheduling,
- atomic confirmation,
- Project → Sequence → Task suppression hierarchy.

---

## 19. API / Persistence Direction

If accepted, canonical Task API and persistence contracts must remove Backlog-specific state.

Potential Task shape change:

```txt
remove placement
remove reviewDate
remove reviewDateSource
```

with ownership-sensitive canonical validation.

A new supporting persisted resource may be required:

```txt
CaptureItem
```

Potential API capabilities:

```txt
create capture
list unresolved captures
resolve capture to Task
resolve capture to Routine
discard capture
```

Exact REST/resource design is deferred until product acceptance.

Promotion must preserve ActionConfirmation/version/idempotency discipline where resolution has consequential canonical effects.

---

## 20. Events / Audit Impact

Backlog-specific event vocabulary should be removed where no longer meaningful:

```txt
PLACEMENT_CHANGED_TO_BACKLOG
KEEP_IN_BACKLOG_WITH_NEW_REVIEW_DATE
```

Task event history should represent actual field/lifecycle changes rather than synthetic placement transitions.

Capture likely requires a minimal event/audit vocabulary such as:

```txt
CAPTURE_CREATED
CAPTURE_RESOLVED
CAPTURE_DISCARDED
```

and linkage from resolution to created canonical entity.

Exact retention and event taxonomy belong to Discussion 019C reconciliation.

---

## 21. Explicit Reopened Decisions

Acceptance of Discussion 026 would explicitly amend the following earlier directions.

### 012A

Reopen:

```txt
No ACTIVE entity may become temporally invisible.
ACTIVE Task requires plannedDate or reviewDate.
Backlog remains valid placement.
```

Replace with responsibility-based temporal reachability.

### 014A

Reopen TaskFields:

```txt
placement
reviewDate
reviewDateSource
plannedDate OR reviewDate validation
```

Replace with ownership-sensitive Task validity.

### 015 / 015A

Reopen:

- Backlog placement semantics,
- Task review checkpoint actions,
- placement-change events,
- ownership-neutral Task temporal rule,
- Task-specific REVIEW_DUE.

### 016 / 016A

Reopen:

- any Task review-due eligibility derived from Task.reviewDate,
- Backlog presentation,
- add unresolved CaptureItem attention without execution-severity inflation.

### 017

Reopen:

- Backlog-specific recommendation/action vocabulary,
- parent review must deterministically include undated direct child Tasks.

### 019A

Reopen canonical Task fields/invariants:

```txt
placement
reviewDate
reviewDateSource
ACTIVE → plannedDate OR reviewDate
```

Add ownership-sensitive plannedDate invariant.

### 019C

Review CaptureItem retention, audit linkage, and removal of Backlog-specific events.

### 020B / 020C

Review Task API/draft validation and CaptureItem resource/command boundaries.

### 022

Implementation sequencing must be rewritten after acceptance.

### 023

Review any Backlog references in rolling context and ensure unfinished parent-owned undated Tasks are still included through canonical parent scope.

### 025

Replace all Backlog sequence semantics with valid parent-owned unscheduled Task semantics.

---

## 22. Proposed Invariants

```txt
Standalone ACTIVE Task
→ plannedDate REQUIRED
```

```txt
Direct Goal-owned ACTIVE Task
→ plannedDate OPTIONAL
```

```txt
Project-owned ACTIVE Task
→ plannedDate OPTIONAL
```

```txt
Parent-owned ACTIVE Task without plannedDate
→ no Task reviewDate
→ no Backlog
→ no independent temporal checkpoint
→ temporal responsibility belongs to direct parent
```

```txt
Task enters Today
→ only when plannedDate = currentLocalDate
```

```txt
Carry
→ only plannedDate old → new
```

```txt
Unresolved CaptureItem
→ persisted supporting record
→ not canonical commitment
→ surfaced in Reconcile
→ excluded from execution severity/failure evidence
```

```txt
Goal / Project reviewDate
→ not required user input in normal creation
→ deterministic system policy when omitted
```

```txt
Goal / Project with targetDate
→ default reviewDate = targetDate
```

```txt
Project without targetDate
→ initial default reviewDate = createdLocalDate + 30 days
```

```txt
Goal without targetDate
→ initial default reviewDate = createdLocalDate + 90 days
```

---

## 23. Open Questions for Claude Review

Claude should review this direction for contradictions, missing states, ambiguity, edge cases, conflicts with accepted Discussions 012–025, and the smallest coherent fixes.

Do not redesign the product from zero. Do not preserve Backlog merely because earlier documents depend on it.

### A. Temporal reachability

1. Is replacing entity-level temporal visibility with commitment-tree-level temporal reachability coherent?
2. Is direct-parent temporal responsibility sufficient for undated Goal/Project-owned Tasks?
3. Are there any paths where an ACTIVE undated child could still become unreachable despite parent review?
4. Should parent targetDate and reviewDate remain separate even when default reviewDate is set equal to targetDate?

### B. Parent review policy

5. Is `reviewDate = targetDate` the correct deterministic default when targetDate exists?
6. Are 30-day Project / 90-day Goal defaults still coherent when no targetDate exists?
7. After `CONTINUE` / `KEEP_WITH_NEW_REVIEW_DATE`, should the next default remain 30/90 days capped by targetDate?
8. Should reviewDate remain hidden from the normal creation form but editable in advanced/detail surfaces?
9. Does an ACTIVE Project always retain a valid parent review boundary under this policy, including when it owns undated Tasks?

### C. Task model

10. Can Task.reviewDate and Task.reviewDateSource be removed completely from MVP?
11. Does any accepted feature still require Task.reviewDate independently from Backlog?
12. Is ownership-sensitive validation sufficient at canonical persistence boundaries?
13. Does completing/dropping an undated parent-owned Task require any special temporal handling beyond ordinary lifecycle rules?

### D. Quick Capture

14. Is `CaptureItem` the smallest coherent persisted supporting record?
15. Should CaptureItem be durable until resolved/discarded, or have any expiry/retention boundary?
16. What minimum fields beyond id/user/title/timestamps/source/version are truly necessary?
17. Should resolution create a new canonical entity and preserve linkage rather than reuse CaptureItem identity?
18. Does unresolved CaptureItem need its own lifecycle/status field, or can resolution metadata/event history suffice?

### E. Reconcile

19. Is unresolvedCaptureCount correctly isolated from execution severity?
20. Should Reconcile badge aggregate captures with other attention counts or expose separate semantics?
21. Do CaptureItems need suppression/dedup behavior if Reconcile also has Project/Goal reviews?
22. Should old CaptureItems ever gain stronger presentation prominence purely because of age without becoming execution debt?

### F. AI Planning

23. Should AI Planning ever emit CaptureItem, or should unresolved AI proposals remain PlanningDraft-only until valid/rejected? Proposed answer: PlanningDraft-only.
24. Is Task draft validation correctly ownership-sensitive after Backlog removal?
25. Does removal of Task reviewDate create any conflict with seven-day detailed planning horizon or Week-N rolling context?

### G. Persistence / API / events

26. Can Task.placement be deleted entirely with no remaining semantic use?
27. Which Backlog events/actions become obsolete and should be explicitly retired?
28. Does CaptureItem need a first-class API resource, or can it be a smaller capture endpoint/resource family while preserving version/idempotency?
29. What audit link is required between CaptureItem resolution and newly created Task/Routine?

### H. Discussion 025 sequences

30. Is replacing `IN_BACKLOG_UNSCHEDULED` with `UNSCHEDULED_PARENT_OWNED` sufficient?
31. Can a valid parent-owned undated Task remain inside a hard dependency sequence without introducing a contradiction?
32. Should CARRY_ALL be allowed to schedule such a Task only after explicit preview/confirmation?
33. Are standalone sequence Tasks correctly forced to remain dated while ACTIVE?

### I. Parent lifecycle

34. Do Goal/Project terminal child-resolution rules already include undated Tasks correctly after Backlog removal?
35. Does parent review need to show all direct ACTIVE undated child Tasks even when the parent itself has no execution overdue state?
36. Are Goal-owned Projects sufficient intermediate review context to avoid duplicate Goal-level review of Project-owned Tasks?

---

## 24. Review Standard

Classify findings as:

```txt
BLOCKING
IMPORTANT
MINOR
```

For each finding include:

```txt
Finding
Why it matters
Conflict / edge case
Smallest coherent fix
Affected earlier discussion(s)
```

Prioritize:

- loss of active commitments,
- hidden temporal invisibility,
- accidental recreation of Backlog under another name,
- unnecessary user questions,
- Task validity contradictions,
- parent-review duplication,
- Quick Capture accidentally becoming execution debt,
- persistence/identity ambiguity,
- Reconcile severity inflation,
- dependency-sequence contradictions,
- smallest coherent MVP model.

---

## 25. Current Proposed Direction Summary

```txt
Backlog
→ removed from MVP
```

```txt
Standalone ACTIVE Task
→ plannedDate required
```

```txt
Parent-owned ACTIVE Task
→ plannedDate optional
→ if undated, direct parent owns temporal resurfacing
```

```txt
Task.reviewDate
→ proposed removal
```

```txt
Goal / Project reviewDate
→ system-managed safety-net checkpoint
→ not required normal user input
```

```txt
if targetDate exists
→ default reviewDate = targetDate
```

```txt
if no targetDate
→ Project +30d
→ Goal +90d
```

```txt
Quick Capture
→ user-facing ordinary fast Add Task flow
→ internal unresolved CaptureItem when no date/parent exists
```

```txt
CaptureItem
→ persisted supporting record
→ not canonical work entity
→ not commitment
→ Reconcile-visible
→ no execution severity/failure evidence
```

```txt
Capture resolution
→ Task if date or parent established
→ Routine if explicitly converted
→ discard otherwise
```

Core principle:

> Preserve every real commitment without forcing every child Task to own an artificial temporal checkpoint, and preserve frictionless capture without pretending unresolved input is already committed work.

---

## 26. Mind Map Impact — NOT YET APPLIED

If accepted, likely Mind Map changes include:

### Product Model

- remove Backlog placement,
- remove Task reviewDate,
- ownership-sensitive Task temporal validity,
- add supporting CaptureItem concept,
- parent-owned undated Tasks are valid.

### MVP Core Loop

```txt
Capture
→ resolve into commitment
→ execute dated Tasks / recurring occurrences
→ review parent-owned undated work through parent
→ Reconcile unresolved execution, reviews, and captures separately
```

### Today

- dated ACTIVE Tasks only,
- no inherited/fake dates.

### Reconcile

- remove Backlog lane/actions,
- add unresolved capture attention,
- parent review includes undated direct child Tasks,
- capture count isolated from execution severity.

### AI Responsibilities

- no Backlog placement proposals,
- ownership-sensitive Task validity,
- no Task reviewDate invention,
- no CaptureItem as AI repair bucket.

### AI Guardrails

- no fake child temporal windows,
- no silent promotion of unresolved capture,
- no capture-as-failure evidence,
- no forced user question for Goal/Project reviewDate.

### Data / Events

- remove placement/backlog events,
- add CaptureItem persistence/audit linkage,
- amend sequence unscheduled classification.

**Do not apply these changes while Discussion 026 remains OPEN.**

---

## 27. Affected Formal Documents — NOT YET APPLIED

If accepted, later reconciliation will be required across at least:

- temporal visibility / reachability specification,
- Task canonical model and validation,
- Goal/Project review default policy,
- PlanningDraft Task schema,
- Task execution / Carry contract,
- Reconcile eligibility/presentation/severity,
- parent review contract,
- canonical persistence invariants,
- event taxonomy and retention,
- API/frontend state contracts,
- structured-output validation,
- implementation plan,
- rolling planning context,
- dependency sequence semantics,
- Mind Map.

No formal document is changed by this open discussion.

---

## 28. Closure Condition

Discussion 026 may be closed only after:

1. Claude review is completed,
2. all Blocking/Important findings are resolved,
3. Backlog removal is proven complete at the product-model level,
4. Task reviewDate removal is validated against all accepted features,
5. commitment-tree temporal reachability is coherent,
6. Goal/Project reviewDate default policy is accepted,
7. Quick Capture identity/persistence/promotion is coherent,
8. Reconcile severity isolation is accepted,
9. Discussion 025 sequence amendments are coherent,
10. Mind Map/formal-document impact is recorded for a later reconciliation pass.

Until then:

```txt
STATUS = OPEN
CLAUDE_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
FORMAL_DOC_RECONCILIATION = PENDING
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

---

# خلاصهٔ فارسی

Discussion 026 حذف کامل Backlog را پیشنهاد می‌کند. در مدل جدید، Task مستقل فعال باید `plannedDate` داشته باشد، ولی Task متعلق به Goal یا Project می‌تواند بدون تاریخ باقی بماند. چنین Taskی نه Backlog است، نه reviewDate مستقل دارد و نه تاریخ یا window مصنوعی از والد به ارث می‌برد. مسئولیت resurfacing آن از طریق review/lifecycle والد مستقیم انجام می‌شود.

اصل temporal visibility از الزام «هر entity فعال باید checkpoint خودش را داشته باشد» به «هیچ commitment فعالی نباید از مسیر deterministic محصول خارج و غیرقابل‌دسترسی شود» تغییر می‌کند.

`Task.reviewDate` و `Task.placement` برای حذف کامل از MVP پیشنهاد شده‌اند. Today همچنان فقط Taskهای ACTIVE با `plannedDate = today` را نشان می‌دهد و Carry فقط تغییر یک plannedDate موجود به plannedDate جدید است.

Goal و Project همچنان review checkpoint دارند، اما reviewDate نباید سؤال اجباری در flow عادی creation باشد. اگر targetDate وجود داشته باشد، default پیشنهادی reviewDate همان targetDate است. اگر targetDate وجود نداشته باشد، default اولیه Project سی روز و Goal نود روز بعد است. این checkpoint یک safety-net سیستم است، نه چیزی که کاربر مجبور باشد هنگام ساخت entity درباره‌اش تصمیم بگیرد.

برای Quick Capture، کاربر همچنان یک flow ساده Add Task با Date اختیاری می‌بیند. اگر نه تاریخ و نه Parent مشخص شود، ورودی به‌عنوان `CaptureItem` durable ذخیره می‌شود، نه canonical Task. CaptureItem commitment نیست، وارد Today و overdue/severity/failure evidence نمی‌شود و در Reconcile برای تعیین تکلیف ظاهر می‌شود. کاربر می‌تواند تاریخ بدهد، به Goal/Project وصل کند، به Routine تبدیل کند یا discard کند.

Discussion 025 نیز در صورت پذیرش 026 amend می‌شود: تمام semantics مربوط به Backlog حذف و با parent-owned unscheduled Task جایگزین می‌شود. Mind Map، formal docs و implementation تا زمان review و بسته‌شدن 026 نباید تغییر کنند.
