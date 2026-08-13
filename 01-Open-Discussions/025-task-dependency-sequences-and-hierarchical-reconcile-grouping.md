# Discussion 025 — Task Dependency Sequences and Hierarchical Reconcile Grouping

## Status

**OPEN — proposed direction, pending Claude review.**

This discussion is **not yet accepted or closed**.

Nothing in this file is authoritative product behavior until review is completed and the resulting decisions are explicitly accepted.

```txt
STATUS = OPEN
CLAUDE_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

Do not update `00-Canvas/Planner-Mindmap.canvas`, formal specifications, implementation plans, or runtime contracts from this file while the discussion remains open.

Primary related accepted discussions:

- [[01-Closed-Discussions/012-core-product-model]]
- [[01-Closed-Discussions/012a-temporal-checkpoint-amendment]]
- [[01-Closed-Discussions/013-ai-planning-entry-and-conversation-flow]]
- [[01-Closed-Discussions/014-ai-planning-output-contract]]
- [[01-Closed-Discussions/016-reconcile-trigger-and-severity]]
- [[01-Closed-Discussions/016a-review-checkpoint-trigger-and-presentation-amendment]]
- [[01-Closed-Discussions/017-ai-reconcile-intelligence-and-actions]]
- [[01-Closed-Discussions/018-action-permissions-trust-and-reversibility]]
- [[01-Closed-Discussions/019a-canonical-data-model-and-invariants]]
- [[01-Closed-Discussions/020b-api-and-frontend-state-contracts]]
- [[01-Closed-Discussions/020c-structured-output-reliability-and-cost-controls]]
- [[01-Closed-Discussions/022-updated-mvp-implementation-plan]]
- [[01-Closed-Discussions/023-persistent-planning-facts-and-rolling-execution-context]]

---

## 1. Problem Statement

The current Task model treats Tasks as independent work items unless they merely share a parent Goal or Project.

That is insufficient for workflows where execution order is meaningful.

Example:

```txt
Project: Build landing page

Task 1 — Research
Task 2 — Wireframe
Task 3 — Visual design
Task 4 — Frontend implementation
Task 5 — QA
```

The user may mean:

```txt
Task 2 should not be treated as executable before Task 1 is complete.
Task 3 should not be treated as executable before Task 2 is complete.
...
```

This is not merely priority.

Priority answers:

```txt
Which Task matters more?
```

Dependency order answers:

```txt
Which Task must come before another Task can meaningfully proceed?
```

A second gap appears in Reconcile.

Example:

```txt
Original schedule:
Day 1  Task 1
Day 2  Task 2
Day 3  Task 3
Day 4  Task 4
Day 5  Task 5
Day 6  Task 6
Day 7  Task 7
Day 8  Task 8
Day 9  Task 9
Day 10 Task 10

Current local day = Day 13
Completed = Task 1–3
Remaining ACTIVE = Task 4–10
```

Showing seven independent overdue Task decisions would be noisy and misleading if those Tasks belong to one ordered execution path.

The user may instead want one coherent decision:

```txt
Carry the remaining sequence from Day 14
```

or:

```txt
Drop the remaining sequence
```

This must coexist with the already accepted Reconcile principle that parent-level issues should suppress redundant child prompts when one higher-level decision is the coherent action unit.

Core requirement:

> Reconcile should surface the highest coherent decision unit that can resolve the problem without making the user decide the same underlying issue repeatedly at lower levels.

---

## 2. Scope

This discussion proposes rules for:

1. distinguishing independent Tasks from Tasks participating in an ordered dependency sequence,
2. representing ordered Task execution without introducing a general-purpose dependency graph,
3. defining when a later sequence Task is blocked by an earlier Task,
4. preserving Task identity and lifecycle semantics,
5. grouping drifted remaining sequence Tasks into one Reconcile decision unit,
6. defining `CARRY_ALL` / grouped replan behavior,
7. defining `DROP_ALL` for remaining sequence Tasks,
8. hierarchical Reconcile suppression across Project → Sequence → Task,
9. deterministic bulk preview / validation / confirmation,
10. deadline and temporal-conflict handling,
11. interaction with Today and blocked Tasks,
12. AI Planning output and dependency-sequence proposal behavior.

### Explicitly out of scope

This discussion does not propose:

- a general DAG dependency engine,
- arbitrary many-to-many Task dependencies,
- cross-Project dependency graphs,
- resource leveling,
- critical-path calculation,
- automatic Project scheduling optimization,
- a new Task lifecycle,
- a new top-level Plan entity,
- a new Reconcile severity model unless required to resolve a contradiction,
- Mind Map changes while this discussion remains open.

---

## 3. Governing Distinction — Priority Is Not Dependency

Task priority and Task dependency are separate concepts.

A higher-priority Task may still be independent.

A dependency sequence expresses execution order.

Proposed MVP distinction:

```txt
Task remains one entity type.

Task may be:
- independent
OR
- a member of one ordered dependency sequence
```

Do not create separate canonical Task entity types merely because behavior differs.

Conceptually:

```txt
Task
- sequenceId?
- sequenceOrder?
```

Rules:

- both fields absent → independent Task,
- both fields present → sequence member,
- sequenceOrder is unique within the sequence,
- one Task belongs to at most one sequence in MVP.

The exact persistence shape for sequence metadata remains open for review.

---

## 4. Dependency Semantics — Proposed HARD Sequence

The user example requires more than visual ordering.

Proposed MVP semantics:

```txt
HARD_SEQUENCE
```

Meaning:

```txt
Task N+1 is not actionable while an earlier required Task in the same sequence remains unresolved.
```

For the basic linear sequence:

```txt
Task 1
↓
Task 2
↓
Task 3
↓
Task 4
```

Task 3 is blocked while Task 2 is still ACTIVE and unresolved.

The exact eligible predecessor states need review.

Initial proposed interpretation:

```txt
COMPLETED predecessor
→ dependency satisfied

DROPPED predecessor
→ does not silently imply dependency satisfied unless sequence consequences are explicitly resolved

ACTIVE predecessor
→ later sequence Task remains blocked
```

A Drop inside the middle of a sequence may require explicit downstream handling rather than guessing that later work is still valid.

This edge case is intentionally open for Claude review.

---

## 5. Sequence Scope

A sequence is expected to exist under one planning parent.

Initial proposed rule:

```txt
all Tasks in one sequence must share the same direct parent scope
```

Possible scopes:

```txt
same Project
OR
same Goal for direct Goal-owned Tasks
```

A sequence must not mix:

```txt
Project A Tasks
+
Project B Tasks
```

or direct Goal Tasks with Project-owned Tasks.

Cross-parent dependencies are out of MVP scope.

---

## 6. Persistence Shape — Open for Review

Two implementation shapes are plausible.

### Option A — Task fields only

```txt
Task
- sequenceId?
- sequenceOrder?
```

Sequence membership is persisted on Tasks.

Sequence-level grouping is derived from shared `sequenceId`.

Advantages:

- no new subordinate entity,
- smaller data-model change.

Open question:

If sequence needs title, provenance, lifecycle metadata, manual reorder history, or independent bulk-operation identity, deriving everything from Task fields may become awkward.

### Option B — subordinate TaskSequence record

Conceptually:

```txt
TaskSequence
- id
- parent scope
- title?
- createdAt
- updatedAt

Task
- sequenceId?
- sequenceOrder?
```

`TaskSequence` would be subordinate structure, not a new top-level work entity and not independently executable.

Discussion 025 does not yet choose between these shapes.

Claude should prefer the smallest coherent model that preserves correctness.

---

## 7. Sequence Order vs Planned Dates

Sequence order and date placement are different concepts.

Example:

```txt
Task A order 1 → Aug 14
Task B order 2 → Aug 14
Task C order 3 → Aug 16
```

This can be valid if Task A and B can be completed in order on the same local date.

Therefore:

```txt
sequenceOrder
≠
plannedDate spacing
```

A sequence does not imply exactly one Task per day.

Replanning should preserve dependency order without inventing artificial one-day spacing.

---

## 8. Today / Execution Behavior

A sequence member may have `plannedDate = today` while still being blocked by an earlier unresolved predecessor.

This requires an explicit execution rule.

Proposed direction:

- blocked Tasks must not appear as ordinary actionable Today checklist items,
- the UI may surface them as blocked context when useful,
- completing a predecessor may make the next planned Task actionable if its temporal placement allows it,
- Reconcile, not Today, remains responsible for resolving accumulated schedule drift.

The exact Today presentation is open for review.

Core invariant:

> Dependency state must not make an ACTIVE Task temporally invisible forever.

A blocked Task still retains its planned/review temporal checkpoint and may participate in Reconcile.

---

## 9. Reconcile Grouping Principle

Reconcile should prefer the highest coherent actionable unit.

Proposed hierarchy:

```txt
Project-level issue
→ show Project

else coherent sequence-level issue
→ show Sequence group

else
→ show individual Task
```

This extends the already accepted adaptive Reconcile principle that a parent-level problem should not produce redundant child prompts when one parent decision is the meaningful action.

### Example

If:

```txt
Project A requires terminal/review resolution
+
Sequence S1 contains seven overdue Tasks
+
Task 8 has a deadline-risk signal
```

and the Project-level decision legitimately governs the underlying work,

Reconcile should avoid presenting:

```txt
Project A card
Sequence S1 card
Task 8 card
```

as three independent obligations.

Instead it should show the Project-level decision and summarize relevant child impact.

---

## 10. Sequence-Level Reconcile Eligibility

A sequence-level prompt is appropriate when multiple remaining Tasks from the same sequence share one coherent drift/replan decision.

Initial candidate conditions:

```txt
same sequenceId
+
multiple ACTIVE unresolved Tasks
+
shared schedule drift / carry need
+
no higher Project-level prompt suppresses them
```

The exact threshold and conflict precedence remain open for review.

A sequence with only one affected Task may fall back to ordinary Task-level Reconcile.

---

## 11. Proposed Sequence-Level Actions

Candidate actions:

```txt
CARRY_ALL
DROP_ALL
REVIEW_INDIVIDUALLY
REVIEW_WITH_AI
```

The first three are core product actions.

`REVIEW_WITH_AI` remains optional assistance over a valid deterministic action surface.

AI does not receive mutation authority merely because the sequence is grouped.

---

## 12. `CARRY_ALL` — Proposed Semantics

`CARRY_ALL` means:

> Re-anchor the remaining eligible ACTIVE sequence schedule while preserving Task identity, sequence order, and relative planned-date offsets.

Example:

Original remaining schedule:

```txt
Task 4 → Day 4
Task 5 → Day 5
Task 6 → Day 6
Task 7 → Day 7
Task 8 → Day 8
Task 9 → Day 9
Task 10 → Day 10
```

User selects:

```txt
newStartDate = Day 14
```

Shift:

```txt
+10 days
```

Result:

```txt
Task 4 → Day 14
Task 5 → Day 15
Task 6 → Day 16
Task 7 → Day 17
Task 8 → Day 18
Task 9 → Day 19
Task 10 → Day 20
```

### Relative-offset preservation

If original schedule is:

```txt
Task 4 → Day 4
Task 5 → Day 4
Task 6 → Day 6
Task 7 → Day 7
```

then re-anchoring to Day 14 yields:

```txt
Task 4 → Day 14
Task 5 → Day 14
Task 6 → Day 16
Task 7 → Day 17
```

Do not normalize to one Task per day.

### Eligible set

Candidate eligible set:

```txt
same selected sequence
+
ACTIVE
+
remaining unresolved segment
```

Do not mutate:

```txt
COMPLETED Tasks
DROPPED Tasks
historical Task identity
sequence order
```

Whether later ACTIVE Tasks beyond an intentionally skipped/dropped middle Task remain eligible requires explicit review.

---

## 13. `DROP_ALL` — Proposed Semantics

`DROP_ALL` applies to the remaining eligible ACTIVE Tasks in the selected sequence segment.

Example:

```txt
Task 1 COMPLETED
Task 2 COMPLETED
Task 3 COMPLETED
Task 4 ACTIVE
...
Task 10 ACTIVE
```

User confirms `DROP_ALL`:

```txt
Task 4–10 → DROPPED
```

Completed history remains unchanged.

This is still a batch of canonical Task lifecycle transitions, not a sequence lifecycle transition unless a future accepted sequence model explicitly adds one.

---

## 14. Bulk Action Safety and Validation

Sequence bulk actions must follow the accepted preview/confirmation discipline.

```txt
build affected set
→ deterministic validation
→ preview consequences
→ explicit user confirmation
→ atomic/bounded application
```

The runtime must not partially apply a grouped replan while silently skipping invalid Tasks.

If any protected Task makes the bulk action invalid, the product should either:

- block the group operation with visible conflict details,
- or explicitly construct a smaller valid affected set before confirmation if existing bulk-action rules permit that behavior.

No hidden partial success.

---

## 15. Deadline and Temporal Conflict Guardrails

`CARRY_ALL` must not blindly shift Tasks past accepted hard temporal boundaries.

Example:

```txt
Task 6 deadline = Day 15
proposed shifted plannedDate = Day 16
```

The user must see the conflict before application.

Candidate behavior:

```txt
bulk preview
→ deadline / temporal validation
→ conflict surfaced
→ user chooses:
   adjust dates
   review individually
   review with AI
   cancel
```

A group action must not weaken Task deadline semantics.

PlanningFact HARD constraints from Discussion 023 may also apply when the sequence belongs to a Goal/standalone Project with applicable confirmed constraints.

---

## 16. Project-Level Suppression vs Sequence-Level Suppression

Sequence grouping must not compete with an already-dominant Project-level decision.

Proposed precedence:

```txt
1. structural/lifecycle Project conflict requiring Project decision
2. Project review/terminal action that coherently governs children
3. sequence-level shared drift decision
4. individual Task decision
```

This is not merely a visual priority rule.

If a higher-level decision changes or resolves the child action surface, lower prompts should be suppressed to avoid duplicate decisions.

If a child has an independent urgent conflict not resolved by the parent action, it may still need explicit disclosure.

Exact conflict precedence remains open for review.

---

## 17. Priority vs Sequence in Reconcile

Task priority must not cause unrelated Tasks to be grouped.

Grouping requires shared dependency-sequence membership and one coherent action.

Examples:

```txt
High-priority Task A
High-priority Task B
```

with no sequence relationship:

```txt
→ independent Reconcile decisions
```

Whereas:

```txt
Task A sequence S1 order 1
Task B sequence S1 order 2
Task C sequence S1 order 3
```

may become one grouped Reconcile unit when shared drift applies.

---

## 18. Reconcile Severity

This discussion proposes grouping/presentation and bulk action semantics, not a new severity model by default.

However, severity must not be accidentally multiplied merely because several child Tasks are suppressed into one sequence prompt.

The accepted severity inputs should continue to derive from underlying deterministic facts.

Presentation grouping must not erase evidence, while duplicate child prompts must not artificially inflate severity merely because the same structural problem is represented at multiple levels.

Claude should verify whether Discussions 016/017 already guarantee this or whether a narrow amendment is required.

---

## 19. AI Planning Impact

AI Planning may propose an ordered Task sequence when the user's intent genuinely requires dependency order.

Example:

```txt
research
→ wireframe
→ implementation
→ QA
```

AI must not infer hard dependency merely because Tasks happen to be listed in one order.

Dependency/sequence semantics must be visible in Draft Review.

User must be able to review or edit the proposed order before approval.

Once approved, sequence membership/order becomes product state and later Week-N planning must respect it.

AI must not silently reorder an accepted sequence during weekly continuation.

A proposed sequence replan may be offered through AI assistance, but canonical mutation still requires explicit review and confirmation.

---

## 20. Manual Task Creation / Editing

Manual Task creation should remain simple.

An independent Task requires no sequence configuration.

If the user chooses to make Tasks dependent, the UI may allow:

```txt
add to sequence
set/reorder sequence position
remove from sequence
```

Removing a Task from the middle of a sequence may affect downstream dependency semantics and therefore requires explicit consequence preview if downstream Tasks exist.

The exact editor UX is out of scope until the domain contract is accepted.

---

## 21. Sequence Mutation Edge Cases

The following require explicit review before closure:

### 21.1 Drop middle Task

```txt
A → B → C
B is DROPPED
```

Does C become:

```txt
blocked
independent from B
requires explicit rewire
```

The system must not guess silently.

### 21.2 Restore dropped Task

Task Detail currently allows restore-in-place for DROPPED Task as an accepted exception.

If a dropped sequence member is restored, dependency consequences for downstream Tasks must remain coherent.

### 21.3 Move Task to Backlog

Backlog is placement, not lifecycle.

If a sequence member is moved to Backlog, what happens to downstream Tasks?

A hard sequence may imply that downstream Tasks cannot remain ordinarily executable while the predecessor has no scheduled execution date.

### 21.4 Carry one Task manually

If the user manually carries only one predecessor, should downstream planned dates stay unchanged and become blocked/drifted, or should the product offer group shift?

The likely MVP answer is to preserve manual authority and surface resulting drift in Reconcile rather than silently cascading dates, but this needs explicit acceptance.

### 21.5 Reorder completed segment

Completed historical Tasks should probably not be casually reordered as if history changed.

The allowed reorder surface for completed vs remaining Tasks requires a precise rule.

---

## 22. Source-of-Truth Discipline

Do not duplicate dependency semantics into free-text AI narrative.

If sequence order is accepted product state, it must be represented structurally.

Do not infer sequence state from:

- Task title prefixes,
- plannedDate ordering alone,
- AI memory,
- parent child-list ordering in UI.

The structural sequence representation is the source of truth.

---

## 23. Proposed Reconcile Example

Current state:

```txt
Project A

Sequence S1
1 Task 1 COMPLETED
2 Task 2 COMPLETED
3 Task 3 COMPLETED
4 Task 4 ACTIVE overdue
5 Task 5 ACTIVE overdue
6 Task 6 ACTIVE overdue
7 Task 7 ACTIVE overdue
8 Task 8 ACTIVE overdue
9 Task 9 ACTIVE overdue
10 Task 10 ACTIVE overdue
```

Today = Day 13.

Instead of:

```txt
7 separate overdue Task cards
```

Reconcile may show:

```txt
Sequence S1
7 remaining Tasks need replanning
original remaining span: Day 4 → Day 10

Actions:
- Carry all from…
- Drop remaining
- Review individually
- Review with AI
```

If the user chooses:

```txt
Carry all from Day 14
```

preview:

```txt
Task 4 Day 4  → Day 14
Task 5 Day 5  → Day 15
Task 6 Day 6  → Day 16
Task 7 Day 7  → Day 17
Task 8 Day 8  → Day 18
Task 9 Day 9  → Day 19
Task 10 Day 10 → Day 20
```

Only after validation and confirmation are Task planned dates updated.

---

## 24. Open Questions for Claude Review

Claude should review this proposed direction for contradictions, missing states, ambiguity, edge cases, and conflicts with accepted Discussions 012–024.

Do **not** redesign the product from zero or reject dependency support merely to preserve MVP simplicity.

### Data model

1. Is `sequenceId + sequenceOrder` sufficient, or does correctness require a subordinate `TaskSequence` record?
2. If a subordinate record exists, what minimum fields are actually necessary?
3. Should sequence membership be allowed for direct Goal-owned Tasks as well as Project-owned Tasks?
4. Can standalone Tasks participate in a sequence, or should sequence require a shared parent?

### Dependency semantics

5. Is HARD linear dependency the correct MVP interpretation?
6. Does `COMPLETED` alone satisfy a predecessor dependency?
7. What should happen when a predecessor is DROPPED?
8. What should happen when a predecessor moves to Backlog?
9. Does restoring a DROPPED predecessor re-block downstream Tasks?
10. Should manual one-Task Carry ever cascade automatically? Proposed answer is no.

### Ordering and history

11. Can remaining ACTIVE Tasks be reordered freely?
12. Can COMPLETED Tasks be reordered, or should historical order remain fixed?
13. What happens if the user removes a Task from the middle of a sequence?
14. Must sequenceOrder remain contiguous, or can gaps exist internally?

### Today / execution

15. Should a blocked Task with plannedDate=today be hidden from ordinary Execution, shown as blocked context, or moved entirely to Reconcile?
16. When a predecessor completes during the day, can the next same-day Task become immediately actionable?
17. How does the no-temporal-invisibility invariant apply to long-blocked Tasks?

### Reconcile grouping

18. What exact conditions qualify multiple Tasks for one sequence-level prompt?
19. Is one affected Task always Task-level rather than sequence-level?
20. What higher-level Project states suppress the sequence card?
21. When must an independent child conflict still be disclosed despite parent/sequence suppression?
22. Does grouping affect severity computation or only presentation/action aggregation?

### Bulk actions

23. Is relative planned-date offset preservation the correct default for `CARRY_ALL`?
24. Should the selected anchor always be the first remaining eligible Task's new plannedDate?
25. What happens to unscheduled/Backlog Tasks inside a sequence during `CARRY_ALL`?
26. Should a deadline conflict block the whole group operation or allow an explicitly reduced affected set?
27. Must grouped changes be one atomic transaction?
28. Which event structure best preserves one user decision with multiple Task mutations without pretending each Task had independent user intent?

### AI Planning

29. When may AI propose hard sequence semantics rather than ordinary ordered suggestions?
30. How should Draft Review expose sequence membership/order without becoming cumbersome?
31. Can AI propose reordering an already accepted sequence during Week-N continuation, or only through an explicit review action?

### Interaction with existing features

32. Does Task restore-in-place conflict with sequence semantics?
33. Does Backlog require a narrow amendment for blocked downstream sequence Tasks?
34. Does Project terminal child-resolution need sequence-aware ordering or preview?
35. Does Discussion 023 PlanningFact context need any dependency-sequence addition, or is canonical sequence state sufficient?

---

## 25. Proposed Review Standard

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

- dependency correctness,
- lifecycle contradictions,
- source-of-truth duplication,
- hidden cascade risk,
- Reconcile suppression mistakes,
- bulk-action authority,
- temporal/deadline conflicts,
- Today visibility,
- recovery from manual edits,
- smallest coherent MVP model.

---

## 26. Current Proposed Direction Summary

Pending review:

```txt
Task remains one canonical entity type.

Independent Task:
sequenceId = null
sequenceOrder = null

Dependent Task:
sequenceId = S
sequenceOrder = N
```

Proposed dependency model:

```txt
linear HARD sequence
Task N+1 waits on required earlier sequence work
```

Reconcile presentation:

```txt
Project-level coherent issue
→ Project prompt

else multi-Task sequence drift
→ Sequence prompt

else
→ Task prompt
```

Sequence bulk actions:

```txt
CARRY_ALL
DROP_ALL
REVIEW_INDIVIDUALLY
REVIEW_WITH_AI
```

`CARRY_ALL` preserves:

```txt
Task identity
sequence order
relative planned-date offsets
```

while changing only eligible remaining ACTIVE Task `plannedDate` values after preview, validation, and explicit confirmation.

Core principle:

> Show the highest coherent decision unit, suppress redundant child prompts, and never silently cascade Task mutations merely because dependency exists.

---

## 27. Mind Map Impact — NOT YET APPLIED

If accepted, likely affected Mind Map areas include:

### Product Model

- optional Task dependency sequence membership,
- explicit separation of priority and dependency,
- possible subordinate TaskSequence structure if review requires it.

### Today

- blocked sequence Task presentation / actionability.

### Reconcile

- hierarchical suppression:
  - Project
  - Sequence
  - Task
- grouped Carry/Drop decisions.

### AI Responsibilities

- propose sequence only when dependency is intended,
- preserve accepted order,
- no silent reordering during continuation.

### AI Guardrails

- no dependency inference from mere list order,
- no hidden cascade of plannedDate changes,
- no mutation without preview/confirmation.

### Data / Events

- sequence membership/order,
- grouped decision/event correlation,
- multi-Task mutation preview and result linkage.

**Do not apply these changes while Discussion 025 remains OPEN.**

---

## 28. Affected Formal Documents — NOT YET APPLIED

If accepted, likely amendments may be required to:

- Task product model / ownership rules,
- Task detail and edit behavior,
- Today execution eligibility,
- Reconcile grouping and suppression rules,
- Reconcile bulk action contract,
- AI Planning output contract,
- canonical persistence model / invariants,
- event / confirmation model,
- frontend state/API contracts,
- Mind Map and implementation plan.

No formal document is changed by this discussion yet.

---

## 29. Closure Condition

Discussion 025 may be closed only after:

1. Claude review is completed,
2. blocking and important findings are resolved,
3. final sequence persistence shape is accepted,
4. dependency/lifecycle edge cases are resolved,
5. Reconcile suppression precedence is accepted,
6. grouped Carry/Drop semantics are accepted,
7. affected prior discussions are explicitly reconciled,
8. Mind Map impact remains recorded for a separate application step.

Until then:

```txt
STATUS = OPEN
CLAUDE_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```
