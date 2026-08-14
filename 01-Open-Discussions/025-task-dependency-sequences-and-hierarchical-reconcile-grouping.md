# Discussion 025 — Task Dependency Sequences and Hierarchical Reconcile Grouping

## Status

**OPEN — Claude review round 1 completed; findings integrated; focused round 2 review pending.**

This discussion is **not yet accepted or closed**.

Nothing in this file is authoritative product behavior until the focused re-review is completed and the resulting direction is explicitly accepted.

```txt
STATUS = OPEN
CLAUDE_REVIEW_ROUND_1 = COMPLETED
ROUND_1_FINDINGS = INTEGRATED
FOCUSED_REVIEW_ROUND_2 = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

Do not update `00-Canvas/Planner-Mindmap.canvas`, formal specifications, implementation plans, or runtime contracts from this file while it remains open.

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
- [[01-Closed-Discussions/019c-events-ai-observability-and-retention]]
- [[01-Closed-Discussions/020b-api-and-frontend-state-contracts]]
- [[01-Closed-Discussions/020c-structured-output-reliability-and-cost-controls]]
- [[01-Closed-Discussions/022-updated-mvp-implementation-plan]]
- [[01-Closed-Discussions/023-persistent-planning-facts-and-rolling-execution-context]]
- [[01-Closed-Discussions/024-multi-time-daily-routine-scheduling-and-occurrence-semantics]]

---

## 1. Problem Statement

The current Task model treats Tasks as independent work items unless they share a parent Goal or Project.

That is insufficient when execution order is semantically required.

Example:

```txt
Task 1 — Research
Task 2 — Wireframe
Task 3 — Visual design
Task 4 — Frontend implementation
Task 5 — QA
```

The user may mean:

```txt
Task 2 is not actionable until Task 1 is completed.
Task 3 is not actionable until Task 2 is completed.
...
```

This is not merely priority.

Priority answers:

```txt
Which Task matters more?
```

Dependency order answers:

```txt
Which Task must be resolved before another Task can meaningfully proceed?
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

Showing seven separate overdue Task decisions is noisy if those Tasks are one ordered execution path.

The user may instead want one coherent decision:

```txt
Carry the remaining sequence from Day 14
```

or:

```txt
Drop the remaining sequence
```

Core requirement:

> Reconcile should surface the highest coherent decision unit that can resolve the underlying issue without forcing redundant lower-level decisions.

---

## 2. Scope

This discussion proposes rules for:

1. distinguishing independent Tasks from dependency-sequence Tasks,
2. representing a linear ordered dependency without a general DAG engine,
3. defining BLOCKED execution semantics,
4. preserving Task identity and lifecycle,
5. grouping drifted sequence Tasks in Reconcile,
6. defining `CARRY_ALL` / `DROP_ALL`,
7. hierarchical suppression across Project → Sequence → Task,
8. deterministic bulk preview / validation / confirmation,
9. deadline and temporal conflict handling,
10. manual scheduling authority inside grouped actions,
11. standalone sequence scope,
12. interaction with Today, Backlog, Drop, Restore, and AI Planning.

### Explicitly out of scope

This discussion does not propose:

- a general DAG dependency engine,
- arbitrary many-to-many dependencies,
- cross-parent dependency graphs,
- critical-path calculation,
- resource leveling,
- automatic Project scheduling optimization,
- a new Task lifecycle,
- a new top-level Plan entity,
- sequence support for Routine,
- a new Reconcile severity system from zero,
- Mind Map changes while this discussion remains open.

---

## 3. Governing Distinction — Priority Is Not Dependency

Task remains one canonical entity type.

A Task may be:

```txt
INDEPENDENT
```

or a member of one linear dependency sequence:

```txt
Task
- sequenceId?
- sequenceOrder?
```

Rules:

- both absent → independent,
- both present → sequence member,
- `sequenceOrder` is unique within the sequence,
- one Task belongs to at most one sequence in MVP.

Dependency is structural product state, not inferred from title, plannedDate order, UI list position, or AI memory.

---

## 4. Proposed HARD Linear Dependency

MVP direction remains a hard linear sequence rather than advisory ordering.

```txt
Task N+1 is BLOCKED while a required earlier Task remains unresolved.
```

Initial state interpretation:

```txt
COMPLETED predecessor
→ dependency satisfied

ACTIVE predecessor
→ downstream remains BLOCKED

ACTIVE predecessor in BACKLOG
→ still unresolved
→ downstream remains BLOCKED

DROPPED predecessor
→ does NOT automatically satisfy dependency
→ downstream remains BLOCKED until explicit structural resolution
```

This prevents the system from guessing that dropping a prerequisite means later work is still valid.

### Dropped predecessor resolution

When a predecessor in the middle of a sequence is DROPPED, downstream Tasks remain blocked until the user explicitly chooses an allowed structural resolution, such as:

```txt
RESTORE_PREDECESSOR
DETACH_OR_REBASE_REMAINING_SEQUENCE
DROP_REMAINING_SEQUENCE
```

Exact naming/UX is still open, but silent satisfaction is rejected.

---

## 5. Sequence Scope — Revised After Claude Review

Round-one review identified a real gap for fully standalone Tasks.

A sequence may exist in exactly one coherent scope:

```txt
A. all members share the same Project
OR
B. all members are direct Tasks of the same Goal
OR
C. all members are standalone Tasks
```

A sequence must not mix scopes.

Forbidden examples:

```txt
Project A Task + Project B Task
Project-owned Task + direct Goal Task
Goal-owned Task + standalone Task
```

Cross-parent dependencies remain out of MVP scope.

This preserves the accepted low-friction ability to create standalone Tasks without forcing an artificial Project merely to express order.

---

## 6. Persistence Direction — MVP

Round-one review favors the smaller model:

```txt
Task
- sequenceId?
- sequenceOrder?
```

No independent `TaskSequence` record is required for MVP unless focused review finds a correctness need.

Important clarification:

`sequenceId` is a durable domain identifier, not an ephemeral UI correlation token and not derived from current Task order.

Product behavior depends on it for:

- blocking,
- grouping,
- reorder scope,
- `CARRY_ALL`,
- `DROP_ALL`,
- audit/event correlation.

The absence of a separate sequence row does not make sequence identity temporary.

---

## 7. Sequence Order vs Planned Dates

Sequence order and planned dates are separate.

```txt
sequenceOrder
≠
plannedDate spacing
```

Example:

```txt
Task A order 1 → Aug 14
Task B order 2 → Aug 14
Task C order 3 → Aug 16
```

This can be valid if A and B can be completed in order on the same date.

A sequence does not imply one Task per day.

### Critical temporal distinction

`plannedDate` means intended execution date.

Dependency state means execution eligibility.

Therefore a Task may legitimately be:

```txt
plannedDate = today
BLOCKED = true
```

This distinction must be preserved in Today and Reconcile.

---

## 8. Today / Execution Behavior

A blocked Task must not appear as an ordinary actionable Today checklist item.

Accepted direction for review:

```txt
plannedDate = today
+
BLOCKED = true
→ not actionable in ordinary Today execution list
```

The product may show blocked context if useful, but must not imply the Task can be completed normally before its dependency is satisfied.

If a predecessor becomes COMPLETED during the same local date, a downstream Task whose `plannedDate = today` may become actionable immediately.

Reconcile remains responsible for accumulated drift rather than Today silently moving dates.

### No temporal invisibility

A blocked ACTIVE Task still keeps its temporal checkpoint.

It may be overdue historically, but blocking affects whether it counts as independently actionable Reconcile backlog, as defined below.

---

## 9. Reconcile Hierarchy

Reconcile should show the highest coherent decision unit.

```txt
Project-level coherent issue
→ show Project

else sequence-level shared issue
→ show Sequence group

else
→ show individual Task
```

This is not only visual priority.

If the higher-level decision truly governs the same child action surface, lower prompts are suppressed.

If an independent urgent child conflict is not resolved by the higher-level action, it must still be disclosed.

---

## 10. Sequence-Level Reconcile Eligibility

A sequence-level prompt is appropriate when:

```txt
same sequenceId
+
multiple relevant remaining Tasks
+
one coherent shared replan/drop decision
+
no dominant Project-level decision suppresses it
```

A sequence with only one actionable affected Task generally falls back to Task-level Reconcile.

The exact threshold remains open for focused review, but mere membership alone does not force grouping.

---

## 11. BLOCKED + Overdue Severity Deduplication

Round-one review identified a concrete double-counting risk.

Example:

```txt
Task 4 ACTIVE + overdue + actionable
Task 5 overdue + BLOCKED by Task 4
Task 6 overdue + BLOCKED by Task 5
Task 7 overdue + BLOCKED by Task 6
```

The system must not count this as four independently actionable overdue decisions.

Accepted proposed rule:

> A BLOCKED descendant Task does not independently increase `actionableBacklogCount` or unresolved actionable-task count merely because its `plannedDate` is in the past.

Its historical overdue state may still exist as deterministic evidence.

Reconcile should count/surface:

```txt
- the actually actionable predecessor decision
and/or
- one coherent sequence-level drift signal
```

rather than each blocked descendant as a separate actionable obligation.

This aligns with Discussion 016's definition that `actionableBacklogCount` counts facts currently requiring an explicit user decision.

---

## 12. Sequence-Level Actions

Candidate actions remain:

```txt
CARRY_ALL
DROP_ALL
REVIEW_INDIVIDUALLY
REVIEW_WITH_AI
```

AI assistance does not gain mutation authority.

All grouped actions require deterministic preview and explicit confirmation.

---

## 13. `CARRY_ALL` Semantics

`CARRY_ALL` means:

> Re-anchor the remaining eligible ACTIVE sequence schedule while preserving Task identity, sequence order, and relative planned-date offsets.

Example:

```txt
Task 4 → Day 4
Task 5 → Day 5
Task 6 → Day 6
...
Task 10 → Day 10
```

User chooses:

```txt
newStartDate = Day 14
```

Result:

```txt
Task 4 → Day 14
Task 5 → Day 15
Task 6 → Day 16
...
Task 10 → Day 20
```

Relative offsets are preserved.

If original spacing is:

```txt
Task 4 → Day 4
Task 5 → Day 4
Task 6 → Day 6
```

then re-anchoring to Day 14 yields:

```txt
Task 4 → Day 14
Task 5 → Day 14
Task 6 → Day 16
```

Do not normalize to one Task per day.

### Base eligible set

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

Backlog/unplanned members require explicit focused-review rules before closure.

---

## 14. Protected Manual Scheduling in `CARRY_ALL`

Round-one review identified that bulk Carry could silently overwrite a newer manual decision on one Task.

That is not allowed.

Before confirmation, preview must classify affected Tasks, for example:

```txt
WILL_SHIFT_NORMALLY
HAS_PROTECTED_MANUAL_SCHEDULE
HAS_TEMPORAL_CONFLICT
```

Example:

```txt
Task 4 → normal group shift
Task 5 → manually scheduled by user to Aug 19 after prior group plan
Task 6 → normal group shift
Task 8 → shifted date would cross deadline
```

A newer explicit manual planned-date decision must never be silently overwritten by `CARRY_ALL`.

The user must explicitly choose whether that protected Task:

```txt
A. joins the new group shift
OR
B. keeps the manually selected date
```

This is a protected override requiring acknowledgement, not automatic exclusion and not silent overwrite.

The exact definition of "newer manual schedule" should use durable mutation/event provenance rather than UI guesswork.

---

## 15. `DROP_ALL` Semantics

`DROP_ALL` applies to the selected remaining eligible ACTIVE sequence segment.

Example:

```txt
Task 1 COMPLETED
Task 2 COMPLETED
Task 3 COMPLETED
Task 4–10 ACTIVE
```

After confirmed `DROP_ALL`:

```txt
Task 4–10 → DROPPED
```

Completed history remains unchanged.

This is a grouped set of canonical Task lifecycle transitions, not a separate sequence lifecycle unless a later accepted model explicitly adds one.

---

## 16. Bulk Action Validation and Atomicity

Grouped actions follow:

```txt
build candidate affected set
→ detect protected manual decisions
→ deterministic temporal/deadline validation
→ preview
→ explicit user choices/confirmation
→ atomic bounded application
```

No hidden partial success.

The runtime must not silently skip invalid Tasks and report a successful full group action.

If the final user-confirmed affected set is reduced, that reduced set must be explicit before commit.

---

## 17. Deadline and Planning Constraint Guardrails

`CARRY_ALL` must not silently move a Task beyond a hard deadline or applicable accepted temporal constraint.

Example:

```txt
Task 6 deadline = Day 15
proposed new plannedDate = Day 16
→ conflict
```

User-visible resolution may include:

```txt
adjust dates
review individually
review with AI
cancel
```

Discussion 023 HARD PlanningFacts also apply when the sequence scope inherits them from a Goal or standalone Project where applicable.

---

## 18. Manual Single-Task Carry

Manual Carry of one Task does not automatically cascade downstream dates.

Accepted direction:

```txt
user manually Carries predecessor
→ preserve manual authority
→ downstream dates remain unchanged
→ downstream may become BLOCKED/drifted
→ Reconcile may later offer sequence-level replan
```

This prevents hidden schedule mutation merely because dependency exists.

---

## 19. Backlog Interaction

Backlog remains Task placement, not lifecycle.

Therefore:

```txt
predecessor moved to Backlog
→ predecessor remains unresolved
→ downstream remains BLOCKED
```

The focused review still needs to decide how `CARRY_ALL` treats a later sequence member that is itself currently in Backlog/unplanned placement.

MVP must not silently schedule such a Task unless the confirmed group action explicitly includes that consequence.

---

## 20. Drop / Restore / Sequence Structure

### Drop middle Task

```txt
A → B → C
B DROPPED
```

C remains blocked until explicit structural resolution.

### Restore

Restore-in-place for a dropped Task remains an accepted Task Detail exception.

Restoring a predecessor does not rewrite sequence history; it restores that Task identity and dependency state must be recalculated from current sequence membership/order.

### Reorder

Completed historical Task order should not be rewritten casually.

Proposed direction:

```txt
COMPLETED historical prefix
→ immutable sequence order

remaining unresolved segment
→ may be reorderable with explicit consequence preview
```

Focused review should validate edge cases where completed and unresolved members are interleaved because of earlier manual edits.

---

## 21. Project vs Sequence Suppression

Proposed precedence:

```txt
1. structural/lifecycle Project conflict requiring Project decision
2. Project review/terminal action that coherently governs children
3. sequence-level shared drift decision
4. individual Task decision
```

A Project card suppresses a Sequence card only when the Project-level action genuinely resolves/governs the same child decision surface.

A sequence or Task conflict that survives the parent decision must remain visible in summary or separate disclosure.

---

## 22. AI Planning Impact

AI may propose a sequence only when the user's intent genuinely expresses prerequisite order.

Mere list order is insufficient.

Draft Review must expose:

```txt
sequence membership
sequence order
hard dependency meaning
```

The user must be able to edit/reject the proposed order before approval.

Once approved, sequence state is canonical Task structure.

Discussion 023 applies:

```txt
sequence has a canonical home
→ NOT PlanningFact
```

Week-N Planning must respect accepted sequence structure and may not silently reorder it.

AI may propose a reordering/replan only through an explicit review path.

---

## 23. Source-of-Truth Discipline

Do not infer sequence state from:

- task titles,
- plannedDate ordering,
- UI list position,
- AI memory,
- parent child-list order.

Structural Task fields are the source of truth.

`sequenceId` is stable identity for the dependency grouping.

`sequenceOrder` is the explicit ordering source.

---

## 24. Round-One Claude Findings — Resolution

Claude found no Blocking issue and three Important findings.

### Finding A — standalone sequences

Resolved by allowing a sequence whose members are all standalone Tasks.

### Finding B — blocked-overdue severity inflation

Resolved by excluding blocked descendants from independent actionable backlog counts while preserving historical overdue evidence and sequence-level signal.

### Finding C — bulk Carry overriding newer manual schedule

Resolved by protected-manual-schedule detection in preview. A newer explicit manual date requires acknowledgement before inclusion in group shift.

Additional accepted review directions:

- `COMPLETED` alone satisfies predecessor dependency,
- `DROPPED` does not silently satisfy dependency,
- Backlog remains unresolved,
- Option A Task fields is preferred for MVP,
- completed historical sequence order should remain immutable,
- sequence state is canonical and must not become PlanningFact.

---

## 25. Focused Round-Two Questions for Claude

Please review only for contradictions, missing states, ambiguity, and smallest coherent fixes. Do not redesign dependency support from zero.

### A. Execution / BLOCKED semantics

1. Is the distinction below sufficient and coherent?

```txt
plannedDate = intended execution date
BLOCKED = execution eligibility gate
```

2. Should a blocked Task with `plannedDate = today` be entirely absent from ordinary Today Execution, or shown in a non-actionable blocked subsection/context?
3. When predecessor completes mid-day, should a downstream Task with `plannedDate = today` become immediately actionable without any extra confirmation?
4. Does excluding BLOCKED descendants from independent `actionableBacklogCount` conflict with any accepted Discussion 016 age/severity rule?

### B. Dropped predecessor / structural resolution

5. Is keeping downstream BLOCKED after predecessor `DROPPED` the smallest safe rule?
6. For MVP, what is the minimal explicit resolution vocabulary after a middle prerequisite is dropped?
7. Should `DETACH_OR_REBASE_REMAINING_SEQUENCE` preserve the same `sequenceId`, create a new sequenceId for the remaining segment, or make the first remaining Task the new sequence root under the same ID?

### C. Reorder semantics

8. Is immutable COMPLETED prefix + reorderable remaining unresolved segment coherent?
9. Must `sequenceOrder` values be contiguous after removals/reorder, or may gaps exist internally while ordering remains deterministic?
10. What happens if a restored dropped Task's old sequenceOrder conflicts with a reordered remaining segment?

### D. Backlog interaction

11. Should a Backlog member inside the remaining sequence be excluded from `CARRY_ALL` by default, or may the explicit grouped action schedule it after clear preview?
12. If a predecessor is moved to Backlog, should Reconcile prefer a sequence-level prompt immediately or wait until ordinary eligibility/drift conditions are met?

### E. Protected manual scheduling

13. What exact event/provenance boundary should define `HAS_PROTECTED_MANUAL_SCHEDULE`?
14. If a protected Task keeps its manual date, how should relative-offset shifting behave for Tasks after it?
   - keep one global shift for all other Tasks,
   - split the remaining sequence into offset segments,
   - or block `CARRY_ALL` until dates are reviewed?
15. Does allowing a reduced explicit affected set preserve hard sequence semantics if one middle Task remains fixed?

### F. Parent suppression

16. Which Project-level states/actions truly suppress sequence-level drift prompts?
17. When a Project-level review chooses `ADJUST_CHILD_EXECUTION`, should that route directly into sequence-aware replanning when relevant rather than showing generic child Task resolution?
18. Can deadline risk on one sequence Task remain separately visible when the sequence card is suppressed by a Project card?

### G. AI Planning

19. What minimum evidence in the user's request is sufficient for AI to propose HARD sequence rather than mere ordered suggestions?
20. Can Week-N AI propose `CARRY_ALL` / sequence re-anchor as a reviewed action, or should grouped replan remain Reconcile-only in MVP?
21. Does sequence creation belong entirely in PlanningDraft work proposals with no additional durable planning-memory type? Proposed answer: yes.

### H. Persistence / events

22. Is `sequenceId + sequenceOrder` on Task sufficient without a TaskSequence row given bulk-action and audit needs?
23. Which event shape best records one grouped decision mutating many Task `plannedDate` values without implying separate independent user decisions?
24. Does restore/reorder require explicit sequence mutation events beyond ordinary Task field-change events for audit coherence?

---

## 26. Current Proposed Direction Summary

```txt
Task remains one canonical entity type.

Independent Task:
sequenceId = null
sequenceOrder = null

Sequence Task:
sequenceId = stable durable ID
sequenceOrder = explicit position
```

Scope:

```txt
same Project
OR same direct Goal
OR all standalone
```

Dependency:

```txt
linear HARD sequence
COMPLETED predecessor → satisfied
ACTIVE/BACKLOG predecessor → unresolved
DROPPED predecessor → explicit structural resolution required
```

Today:

```txt
plannedDate may be today
but BLOCKED means not ordinarily actionable
```

Reconcile:

```txt
Project coherent issue
→ Project prompt

else shared sequence drift
→ Sequence prompt

else
→ Task prompt
```

Severity:

```txt
BLOCKED descendants
→ not independently counted as actionable backlog
```

Bulk actions:

```txt
CARRY_ALL
DROP_ALL
REVIEW_INDIVIDUALLY
REVIEW_WITH_AI
```

`CARRY_ALL` preserves:

```txt
Task IDs
sequence order
relative planned-date offsets
```

while protecting:

```txt
hard deadlines
PlanningFact constraints
newer explicit manual scheduling decisions
```

Core principle:

> Show the highest coherent decision unit, preserve explicit user authority, and never silently cascade or overwrite Task scheduling merely because dependency exists.

---

## 27. Mind Map Impact — NOT YET APPLIED

If later accepted, likely affected areas include:

### Product Model

- optional linear Task dependency sequence,
- stable `sequenceId`,
- explicit `sequenceOrder`,
- standalone sequence scope,
- separation of dependency from priority.

### Today

- blocked planned Tasks are not ordinary actionable execution.

### Reconcile

- Project → Sequence → Task suppression hierarchy,
- blocked-overdue dedupe,
- sequence-level Carry/Drop,
- protected manual-date handling.

### AI Responsibilities

- propose hard sequence only with real prerequisite intent,
- expose sequence in Draft Review,
- preserve accepted order.

### AI Guardrails

- no hard dependency inference from mere list order,
- no hidden cascade,
- no silent overwrite of recent manual scheduling,
- no PlanningFact duplication for sequence state.

### Data / Events

- stable sequence membership/order,
- grouped decision correlation,
- multi-Task mutation preview/result linkage.

**Do not apply while Discussion 025 remains OPEN.**

---

## 28. Affected Formal Documents — NOT YET APPLIED

If accepted, likely amendments may be required to:

- Task product model / invariants,
- Today execution eligibility,
- Reconcile eligibility/severity derivation,
- Reconcile grouping/suppression,
- bulk action contract,
- Backlog interaction,
- Task Drop/Restore behavior,
- AI Planning output contract,
- canonical persistence model,
- event/confirmation model,
- API/frontend contracts,
- Mind Map and implementation plan.

No formal document is changed by this discussion yet.

---

## 29. Closure Condition

Discussion 025 may be closed only after:

1. focused Claude round-two review is completed,
2. any new Blocking/Important findings are resolved,
3. blocked Today behavior is accepted,
4. dropped predecessor structural resolution is accepted,
5. Backlog interaction is accepted,
6. protected manual scheduling behavior inside `CARRY_ALL` is accepted,
7. parent-vs-sequence suppression is accepted,
8. persistence/event implications are coherent,
9. Mind Map impact remains recorded for separate application.

Until then:

```txt
STATUS = OPEN
CLAUDE_REVIEW_ROUND_1 = COMPLETED
ROUND_1_FINDINGS = INTEGRATED
FOCUSED_REVIEW_ROUND_2 = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

---

# خلاصهٔ فارسی

Discussion 025 هنوز باز است. سه finding دور اول Claude ادغام شده‌اند: sequence برای Taskهای کاملاً standalone هم مجاز است، Taskهای blocked و overdue به‌صورت مستقل severity/actionable backlog را چند برابر نمی‌کنند، و `CARRY_ALL` حق ندارد تصمیم دستی جدیدتر کاربر روی تاریخ یک Task را بی‌صدا overwrite کند.

مدل MVP همچنان Task entity جدید یا dependency graph عمومی نمی‌سازد. Task می‌تواند مستقل باشد یا `sequenceId + sequenceOrder` داشته باشد. sequence یک dependency خطی سخت است. فقط predecessor COMPLETED dependency را satisfy می‌کند؛ Backlog unresolved است و Drop وسط sequence نیاز به تصمیم ساختاری صریح دارد.

`plannedDate` و actionability از هم جدا شده‌اند: ممکن است Task برای امروز برنامه‌ریزی شده باشد ولی به‌دلیل dependency BLOCKED باشد. این Task نباید مثل Task معمولی در Today actionable دیده شود.

Reconcile باید در بالاترین سطح تصمیم منسجم کار کند: Project، وگرنه Sequence، وگرنه Task. `CARRY_ALL` ترتیب و فاصله‌های نسبی را حفظ می‌کند، اما deadlineها، PlanningFactهای سخت و تصمیم‌های دستی جدیدتر را قبل از اعمال محافظت و نمایش می‌دهد.

سند اکنون برای یک focused Claude round-two review آماده است و Mind Map/implementation هنوز نباید تغییر کند.
