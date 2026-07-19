# Discussion 017 — AI Reconcile Intelligence and Actions

## Status

Open for GPT × Claude review.

This revision incorporates Claude findings F1–F6 where accepted, preserves the bounded metric-driven Reconcile model, and leaves exactly two cross-model product questions open for final review:

1. whether Reconcile may present a deterministic Goal intent-confirmation question after meaningful absence and stale descendant execution
2. whether every active Goal, Project, and Task must have a future temporal checkpoint

Related accepted discussions:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/014-ai-planning-output-contract]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/016-reconcile-trigger-and-severity]]

---

## 1. Scope

Discussion 016 determines **when** Reconcile appears and **how strongly** it is presented.

Discussion 017 determines what happens after Reconcile is opened:

- which facts are handled deterministically
- which metrics may be evaluated
- which bounded rules may match
- how items are grouped
- which recommendation types are allowed
- which actions remain user-owned
- when Task-level facts may justify a Routine or Project-level recommendation
- how rationale and evidence are shown
- what Reconcile may pass to Adaptive Planning
- whether a bounded Goal continuation check belongs in Reconcile
- how temporally unplaced active entities should be prevented from becoming invisible backlog

Core question:

```txt
Given accepted execution facts,
which limited and explainable actions may the product suggest,
without guessing about the user or silently rewriting commitments?
```

---

## 2. Explicitly Out of Scope

This discussion does not define:

- Reconcile trigger or severity — Discussion 016
- open-ended semantic interpretation
- personality, motivation, discipline, mood, lifestyle, or psychological diagnosis
- unrestricted duplicate detection by language similarity
- automatic Goal achievement or abandonment
- automatic Project completion or stopping
- automatic Routine stopping or continuation creation
- database schema, indexes, or event persistence — Discussion 019
- provider, prompt, API, or runtime architecture — Discussion 020
- exact validation plan — Discussion 021
- implementation sequencing — Discussion 022
- full Adaptive Planning algorithm or weekly-plan generation
- visual styling, animation, or responsive layout

The MVP intentionally prefers a small number of explicit rules over broad AI coverage.

---

## 3. Accepted Product Hierarchy

Accepted ownership from Discussion 012:

```txt
Project → one Goal or standalone
Task → one Goal, one Project, or standalone
Routine → one Goal, one Project, or standalone
RoutineOccurrence → exactly one Routine
```

Task and Routine may not belong to Goal and Project simultaneously.

Reconcile must support:

- Project-owned Tasks and Routines
- direct Goal-owned Tasks and Routines
- standalone Tasks and Routines

Stable ownership is always applied before secondary grouping.

---

## 4. Governing Principle: No Free Interpretation

Accepted direction from Mahdi:

> AI must not guess. Approved metrics, rule combinations, problem statements, and allowed recommendation types must be explicit.

The governing pipeline is:

```txt
deterministic execution facts
→ stable ownership grouping
→ approved metric calculation
→ bounded rule matching
→ predefined recommendation options
→ explicit user decision
```

AI is not an open-ended behavioral analyst.

Its allowed MVP role is:

```txt
structured evaluator
+ explanation generator
+ constrained recommendation presenter
```

It may not:

- invent a new problem category
- infer that the user lacks motivation
- infer emotional state or discipline
- infer that a Goal is no longer meaningful
- infer low capacity from absence
- infer failure from raw MISSED counts
- infer future availability from past behavior
- recommend an action outside the accepted rule action set
- change canonical entities without confirmation

---

## 5. Data-Only Intelligence Boundary

### 5.1 Allowed Inputs

Reconcile intelligence may consume only canonical entities, recorded execution events, and deterministic derived metrics.

Examples:

```txt
Task.plannedDate
Task.createdAt
Task.lastMeaningfulInteractionAt
Task.carryCount
Task.status
Task.deadline
Task.protection flags
Routine recurrence
RoutineOccurrence scheduled date
RoutineOccurrence DONE | MISSED
Project and Goal lifecycle
ownership relationships
explicit user actions already recorded by the product
absence and observation boundaries
```

AI must consume these structured facts rather than recreate them from prose.

### 5.2 Prohibited Questions and Inputs

AI must not ask causal, emotional, motivational, capacity, or future-prediction questions.

Examples explicitly forbidden:

```txt
Why did you not do this?
What stopped you?
Were you unmotivated?
When will you feel better?
When do you think you will have enough time?
Was your schedule too busy?
```

Free-text user explanations must not be used to:

- match a Reconcile intelligence rule
- create or modify a metric
- infer a cause
- infer motivation, capacity, mood, or lifestyle
- appear in rationale as an AI-derived fact

Optional user-authored notes may be stored as notes, but they remain outside Reconcile rule evaluation.

### 5.3 Bounded AI Responsibilities

AI may:

- turn matched rule facts into concise human-readable rationale
- present only the recommendation options attached to the matched rule
- explain deterministic consequences already established by product rules
- summarize a resolved Reconcile session

AI may not select a recommendation type before a deterministic rule match exists.

### 5.4 Open Boundary: Intent Confirmation

One question remains intentionally open:

> May Reconcile present a deterministic, closed-form Goal continuation check after meaningful absence and stale descendant execution?

This is not a causal question. It does not ask why work was missed or predict future capacity.

Current proposal for Claude review:

```txt
Goal.status = ACTIVE
AND meaningful absence threshold matched
AND one or more descendant Tasks or Routines contain actionable stale execution
AND no later explicit Goal reaffirmation exists
→ present Goal continuation check
```

Proposed fixed options:

```txt
CONTINUE_GOAL
ABANDON_GOAL
REVIEW_LATER
```

Proposed constraints:

- this is a deterministic product action, not an AI interpretation
- AI must not infer the answer from execution data
- no free-text explanation is required or analyzed
- `CONTINUE_GOAL` records explicit reaffirmation only
- `ABANDON_GOAL` enters the accepted Goal terminal child-resolution flow
- `REVIEW_LATER` changes presentation state only and does not create `PAUSED`
- the Goal target date does not need to have passed
- the trigger comes from stale descendant execution after absence, not proof that the Goal failed

This proposal is not accepted until Claude reviews its boundary against the no-guessing principle.

### 5.5 Recommendation Contract

Every AI-backed suggestion must include:

```txt
matchedRuleId
affectedEntityIds
observedMetrics
exact problem statement
allowedActions[]
consequence summary
evidence quality
```

The explanation must remain understandable without exposing implementation jargon.

---

## 6. Grouping Model

### 6.1 Stable Ownership First

```txt
Project-owned work → under that Project
Goal-owned work → under that Goal
standalone work → standalone section
RoutineOccurrences → under their Routine
```

AI may not flatten Project-owned work into direct Goal work.

### 6.2 Rule-Based Grouping

Within an ownership group, items may be grouped only by accepted rule relationships:

- repeated Carry on the same Task
- stale dated Tasks
- common deterministic deadline risk
- the same Routine observation rule
- the same structural conflict
- the same correction review

### 6.3 No Free Semantic Grouping

MVP excludes:

- duplicate inference from wording alone
- broad topic similarity grouping
- grouping by inferred intent
- grouping by assumed shared outcome

### 6.4 User Control

The user may:

- remove an item from a proposed rule group
- review an item individually
- reject the recommendation for the whole group
- apply an allowed action only to selected items

MVP does not require arbitrary user-created merge groups inside Reconcile.

---

## 7. Protected Items

A protected Task must not receive destructive or de-prioritizing bulk suggestions.

A Task is protected when an accepted deterministic condition is true, including:

- user explicitly marked it protected
- hard deadline exists
- legal, financial, health, or external obligation flag exists
- it blocks another accepted dependency
- it belongs to an active parent marked high priority
- user recently reaffirmed it through an explicit product action
- it is currently in progress

Protected-item rules:

- no automatic or bulk Drop suggestion
- no silent Backlog move
- no suggestion that conceals deadline risk
- rationale identifies the protection reason
- available actions emphasize explicit scheduling, splitting, or direct review

Exact field representation is deferred to Discussion 019.

---

## 8. Initial MVP Intelligence Rules

The MVP begins with a deliberately small rule catalog.

### R1 — Repeated Carry Task

Trigger:

```txt
Task.status = ACTIVE
carryCount >= 2
```

Allowed actions:

```txt
REPLAN_TO_EXPLICIT_DATE
MOVE_TO_BACKLOG
SPLIT_TASK
DROP_TASK, only when not protected and no deadline risk
KEEP_UNCHANGED
```

Constraints:

- first Carry alone does not trigger R1
- the rule applies to the same Task identity
- it does not diagnose motivation or capacity
- it does not automatically change Project or Goal state

### R2 — Stale Dated Task

Trigger:

```txt
Task.status = ACTIVE
Task.plannedDate exists
staleAgeDays >= staleAgeThreshold
no blocking structural conflict
```

Where:

```txt
staleAgeDays = currentLocalDate - authoritative plannedDate
```

Allowed actions:

```txt
REPLAN_TO_EXPLICIT_DATE
MOVE_TO_BACKLOG
DROP_TASK, only when protected = false and deadlineRisk = false
KEEP_UNCHANGED
```

Current proposal:

```txt
staleAgeThreshold = 7 local days
```

Clarification accepted from Claude:

```txt
R2 staleAgeThreshold
≠ Discussion 016 severity age threshold
```

The values may currently match but represent independent product rules and may be tuned separately.

Undated Tasks are not silently included in R2. Their treatment remains part of the open temporal-checkpoint question in Section 17.

### R3 — Deadline Risk

Trigger:

```txt
deadline exists
Task is unresolved
remainingExecutionWindow <= accepted risk threshold
```

Allowed actions:

```txt
SCHEDULE_NEXT_ACTION
SPLIT_TASK
REVIEW_CONFLICTING_TASKS
KEEP_WITH_ACKNOWLEDGED_RISK
```

Constraints:

- no automatic Drop recommendation
- no Backlog recommendation that conceals the deadline
- explanation shows the real deadline and remaining time
- newly detected real deadline risk may use the same-day retrigger exception from Discussion 016

### R4 — Routine Mismatch Candidate

R4 may match only from observations after the accepted first-week calibration boundary from Discussion 014.

Canonical evidence calculation:

```txt
observedOccurrenceCount
missedRatio
evidenceQuality
```

Trigger:

```txt
Routine.status = ACTIVE
postCalibrationObservedOccurrenceCount >= minimumObservedOccurrences
missedRatio >= missedRatioThreshold
evidenceQuality = SUFFICIENT
```

Current proposals:

```txt
minimumObservedOccurrences = 6
missedRatioThreshold = 0.5
```

Accepted clarifications from Claude:

- calibration occurrences remain historical but do not count toward R4 minimum evidence
- absence contamination is not a second independent gate
- one canonical evidence-quality calculation produces `SUFFICIENT`, `LIMITED`, or `ABSENCE_CONTAMINATED`
- R4 recommendations are allowed only when evidence quality is `SUFFICIENT`

Allowed actions:

```txt
CHANGE_ROUTINE_DAYS
REDUCE_ROUTINE_FREQUENCY
REVIEW_ROUTINE_TIME
STOP_ROUTINE
KEEP_ROUTINE_UNCHANGED
```

Constraints:

- no lifestyle inference
- raw MISSED count alone is insufficient
- automatically reconstructed absence facts do not establish mismatch
- stopping requires explicit user confirmation
- later Resume creates a new continuation Routine under Discussion 015

### R5 — Project Execution Overload Candidate

R5 is intentionally narrow and uses both absolute and proportional coverage.

Trigger:

```txt
Project.status = ACTIVE
observed across at least minimumObservedPeriods
AND one of:

A)
activeTaskCount >= 4
AND repeatedCarryTaskCount >= 2

OR

B)
activeTaskCount >= 2
AND repeatedCarryTaskCount >= 2
AND repeatedCarryTaskCount / activeTaskCount >= 0.5
```

Current proposal:

```txt
minimumObservedPeriods = 2
```

The proportional branch covers small Projects without escalating a one-Task Project into a Project-level diagnosis.

Allowed actions currently supported by accepted fields:

```txt
MOVE_SELECTED_TASKS_TO_BACKLOG
SPLIT_SELECTED_TASKS
KEEP_PROJECT_UNCHANGED
```

`EXTEND_PROJECT_TARGET_DATE` is removed from the accepted action set for now because Discussion 014 has not accepted a canonical `Project.targetDate` field. It may return only if the temporal-checkpoint question formally introduces that field.

Constraints:

- no vague “review your Project” recommendation
- exact affected counts and periods must be shown
- no automatic Project completion
- no automatic Project stop
- no child detachment
- no Goal-level conclusion

### R6 — Structural Lifecycle Conflict

Examples:

- Project terminal action attempted with active child Tasks
- Goal terminal action attempted with active children

This is deterministic, not an AI discovery rule.

Allowed behavior:

- explain the exact conflicting children
- present only lifecycle actions accepted in Discussion 015
- show the consequence of each action

Project child options may include:

```txt
COMPLETE_SELECTED_TASKS
DROP_SELECTED_TASKS
DETACH_OR_REPARENT_SELECTED_TASKS
CANCEL_PARENT_TRANSITION
```

AI explains but does not choose.

---

## 9. Action Taxonomy

### 9.1 Task Actions

```txt
COMPLETE_TASK
KEEP_UNCHANGED
REPLAN_TO_EXPLICIT_DATE
MOVE_TO_BACKLOG
DROP_TASK
SPLIT_TASK
```

### 9.2 RoutineOccurrence Actions

```txt
MARK_DONE
KEEP_MISSED
CORRECT_OCCURRENCE
ADD_OPTIONAL_CONTEXT
```

Occurrence restrictions:

- no Carry
- no Replan of the historical occurrence
- scheduled local date does not change
- optional context is not an intelligence metric

### 9.3 Routine Actions

```txt
CHANGE_ROUTINE_DAYS
REDUCE_ROUTINE_FREQUENCY
REVIEW_ROUTINE_TIME
STOP_ROUTINE
KEEP_ROUTINE_UNCHANGED
```

Routine edits obey prospective effective-date rules from Discussion 015.

### 9.4 Project Actions

Accepted normal Reconcile actions:

```txt
MOVE_SELECTED_TASKS_TO_BACKLOG
SPLIT_SELECTED_TASKS
KEEP_PROJECT_UNCHANGED
```

Project terminal actions are not normal AI recommendations.

When the user independently initiates a Project terminal transition, Reconcile supports the required child-resolution flow from Discussion 015.

### 9.5 Goal Actions

Normal MVP Reconcile does not recommend:

```txt
ACHIEVE_GOAL
ABANDON_GOAL
```

It never infers Goal outcome achievement from completed work.

A possible deterministic Goal continuation check remains open in Section 17 and must not be treated as accepted normal AI recommendation behavior yet.

---

## 10. Bulk Suggestions

### 10.1 Bulk Eligibility

A bulk suggestion is allowed only when:

- every selected item matches the same accepted rule
- every selected item supports the same allowed action
- consequences are local and visible
- protected items are excluded where required
- selected items are fully previewed
- the user explicitly confirms

### 10.2 Bulk-Safe Candidates

```txt
MOVE_SELECTED_TASKS_TO_BACKLOG
REPLAN_SELECTED_TASKS_TO_ONE_EXPLICIT_DATE
KEEP_SELECTED_ITEMS
```

`DROP_SELECTED_TASKS` may be allowed only when every item:

- is unprotected
- has no deadline risk
- is individually visible in preview
- is explicitly selected

### 10.3 Parent-Empty Consequence Disclosure

Accepted correction from Claude F6:

Before bulk Drop or Backlog confirmation, the product must disclose when the selected action will leave a Project or Goal with no active executable child work.

Example meaning:

```txt
This action will leave Project “Build Portfolio”
with no active Tasks or Routines.
The Project will remain ACTIVE until you separately complete or stop it.
```

The product must not silently complete, stop, achieve, or abandon the parent.

### 10.4 Item-Level Review Required

```txt
COMPLETE_TASK
SPLIT_TASK
DROP_TASK when contextual risk exists
CORRECT_OCCURRENCE
STOP_ROUTINE
DETACH_OR_REPARENT_TASK
any Project or Goal terminal decision
```

### 10.5 Forbidden Autonomous Actions

AI must never autonomously:

- complete or drop work
- stop a Routine
- complete or stop a Project
- achieve or abandon a Goal
- merge or delete entities
- detach or reparent children
- change deadlines or temporal checkpoints

---

## 11. Escalation Model

Escalation means matching a bounded parent-level rule. It does not mean vague advice.

### Task to Routine

Allowed only when R4 matches.

The product states:

- post-calibration observed occurrence count
- missed ratio
- canonical evidence quality
- exact allowed Routine changes

### Tasks to Project

Allowed only when R5 matches.

Example:

> Two of three active Tasks in this Project have each been carried at least twice across two observed periods.

Forbidden substitute:

> Consider reviewing this Project.

### Project to Goal

MVP performs no interpretive Goal escalation.

It may report deterministic context but may not infer:

- the Goal no longer matters
- the Goal was achieved
- the Goal should be abandoned
- Project difficulty proves Goal mismatch

The bounded continuation-check proposal remains open separately.

### Standalone Work

Standalone Tasks and Routines use the same Task and Routine rules without parent escalation.

The absence of a parent must not cause AI to invent one.

---

## 12. Recommendation Frequency

Accepted qualitative contract:

```txt
Recommendations must be limited, prioritized, consolidated, and chunked.
```

Exact numerical limits by severity are deferred to UX and validation specifications.

Inside one group:

- one exact problem statement
- only the rule's allowed actions
- no repeated rationale per item
- simple cases use deterministic quick actions without AI-backed recommendation text

---

## 13. Evidence Quality and Rationale

Canonical evidence quality:

```txt
SUFFICIENT
LIMITED
ABSENCE_CONTAMINATED
```

Evidence quality is calculated once from accepted observations and absence boundaries.

A rule may generate a recommendation only when its required evidence quality is met.

Every recommendation must answer:

```txt
What exact rule matched?
Which recorded facts support it?
Why are these actions allowed?
What deterministic consequence follows?
```

Prohibited rationale includes:

- you seem unmotivated
- this Goal no longer fits you
- your productivity is low
- this Routine does not match your lifestyle
- you are taking on too much, unless R5 exactly matched
- you will probably have more capacity later

---

## 14. Consequence Disclosure

### Stop Routine

Disclose:

- future occurrences stop after the accepted boundary
- historical occurrences remain
- later Resume creates a new continuation Routine

### Complete or Stop Project

When user initiated, disclose:

- active child Tasks requiring resolution
- active Project-owned Routines that will stop
- same-day occurrence behavior
- future occurrence ineligibility
- no active child may remain under terminal Project

### Goal Terminal Actions

When user initiated, disclose:

- active Projects, Tasks, and Routines requiring resolution
- Goal outcome must be confirmed by the user
- execution completion does not prove outcome achievement

### Bulk Child Removal

Disclose when the parent will remain active with no active executable child work.

---

## 15. Reconcile to Adaptive Planning Handoff

Reconcile may output structured resolved evidence:

- Tasks completed
- Tasks carried or replanned
- Tasks moved to Backlog
- Tasks dropped
- Routine changes confirmed
- recommendations rejected
- evidence quality
- absence-contaminated periods
- explicit Goal reaffirmation, only if the open continuation-check proposal is accepted

Allowed follow-up:

- record evidence
- summarize session
- offer entry into a separate planning review

Not allowed inside Reconcile:

- silently regenerate the weekly plan
- change future workload without approval
- infer long-term capacity from one session
- treat absence as confirmed failure

---

## 16. Open Question A — Goal Continuation Check

### Problem

A Goal may remain `ACTIVE` and have no expired Goal date, while the user is absent for a meaningful period and descendant Tasks or Routine execution become stale.

Reconcile needs a way to avoid two bad outcomes:

```txt
keep assuming the Goal remains actively pursued forever
OR
infer from execution failure that the Goal should be abandoned
```

### Current Proposal

Allow a deterministic intent-confirmation checkpoint:

```txt
Do you still want to continue Goal “X”?

CONTINUE_GOAL
ABANDON_GOAL
REVIEW_LATER
```

This check:

- asks no causal or emotional question
- requests no explanation
- consumes no free-text answer
- records only an explicit lifecycle intention
- may trigger from absence plus stale actionable descendant execution
- does not require the Goal date to have passed
- does not claim failure

### Questions for Claude

1. Does this violate the accepted no-question/no-guessing boundary, or is it a legitimate deterministic lifecycle confirmation?
2. Should the trigger require only absence plus stale descendants, or also a scheduled Goal review checkpoint?
3. Should `ABANDON_GOAL` be offered directly, or should it navigate to a separate Goal terminal flow?
4. Is `REVIEW_LATER` sufficient without introducing `PAUSED`?
5. What minimum guardrail prevents this check from becoming repetitive or punitive?

This question remains open.

---

## 17. Open Question B — Temporal Checkpoints for Active Entities

### Problem

Current accepted model allows:

- Task without `plannedDate`
- Project without canonical target date
- Goal without canonical target date

Long-lived undated entities may create:

- invisible or forgotten work
- a permanent undated backlog
- unclear Reconcile timing
- unclear commitment state
- extra mental load from a separate undated area
- difficulty distinguishing intentional backlog from abandoned placement

Task progress percentage is not the primary concern because Goal progress is not an automatically inferred percentage. The core problem is temporal visibility and reviewability.

### Current Proposal

Adopt the principle:

```txt
No ACTIVE entity may be temporally invisible.
```

Possible invariant:

```txt
Task:
plannedDate OR reviewDate

Project:
targetDate OR reviewDate

Goal:
reviewDate required
targetDate optional

Routine:
recurrence and effective start provide next occurrence
```

Alternative uniform representation:

```txt
Every ACTIVE entity has nextTemporalCheckpoint.
```

Possible checkpoint meanings:

```txt
Task plannedDate → execution date
Task reviewDate → placement decision checkpoint
Project targetDate → intended completion boundary
Project reviewDate → scope/status checkpoint
Goal targetDate → optional desired outcome boundary
Goal reviewDate → continuation and direction checkpoint
Routine next occurrence → derived from recurrence
```

### Proposed Display Semantics

```txt
Today
→ Tasks with plannedDate = current local date

Upcoming
→ Tasks with future plannedDate

Overdue
→ unresolved Tasks whose plannedDate passed

Upcoming Reviews / Reconcile checkpoints
→ active entities whose reviewDate is approaching or passed
```

No permanent unbounded `Undated` page is required if every active entity has a review checkpoint.

### Proposed Reconcile Semantics

```txt
Task reviewDate passed
→ placement decision checkpoint, not overdue execution

Project reviewDate passed
→ bounded Project status/scope checkpoint

Goal reviewDate passed
→ possible Goal continuation check
```

### Cross-Discussion Impact if Accepted

This would amend or extend:

- Discussion 012: active entity temporal invariant
- Discussion 014: PlanningDraft fields and validation
- Discussion 015: Today, overdue, and review-checkpoint semantics
- Discussion 016: Reconcile eligibility and severity inputs
- Discussion 017: bounded checkpoint actions
- Discussion 019: persistence and events
- Discussion 020: runtime validation

### Questions for Claude

1. Is requiring a future temporal checkpoint for every active entity coherent, or does it over-constrain legitimate long-term work?
2. Should `reviewDate` be required only when an execution/target date is absent, or always required for Goal and Project?
3. Should Task `plannedDate = null` remain valid when a `reviewDate` exists?
4. Does `nextTemporalCheckpoint` need to be one canonical field or a derived abstraction over entity-specific dates?
5. How should checkpoint expiry affect Reconcile severity without confusing review debt with execution overdue?
6. Does this proposal belong inside Discussion 017, or should 017 only record the requirement and reopen 012/014/015 through an explicit amendment?
7. Is a separate explicit Backlog placement still needed, or can `reviewDate` fully prevent the undated graveyard problem?

This question remains open.

---

## 18. Scenario Checks

### One Overdue Task

```txt
ageDays = 1
carryCount = 0
no deadline risk
```

Expected:

- deterministic quick actions
- no AI-backed interpretation required

### Task Carried Twice

Expected:

- R1 matches
- exact Carry count shown
- limited actions
- no motivation or capacity claim

### Protected Old Task

Expected:

- R2 may identify stale age
- destructive options excluded
- R3 deadline protection controls allowed actions

### Routine During Absence

```txt
10 reconstructed MISSED occurrences
little observed interaction
```

Expected:

- evidence quality `ABSENCE_CONTAMINATED`
- no R4 recommendation
- no frequency reduction from raw missed count

### Post-Calibration Routine Pattern

```txt
8 post-calibration observed occurrences
5 MISSED
evidenceQuality = SUFFICIENT
```

Expected:

- R4 matches
- exact ratio shown
- predefined Routine actions only

### Small Project Carry Pattern

```txt
2 active Tasks
2 repeated-Carry Tasks
pattern observed across 2 periods
```

Expected:

- proportional R5 branch matches
- exact 2 of 2 pattern shown
- no vague Project review advice

### Bulk Drop Empties Project

Expected:

- full selected-item preview
- explicit warning that Project will remain ACTIVE with no executable children
- no automatic Project stop or completion

### Month-Long Absence Under Active Goal

```txt
Goal ACTIVE
absence threshold matched
several stale descendant Tasks or Routine facts
Goal date has not passed
```

Expected while Question A remains open:

- no inferred abandonment
- no causal or emotional question
- candidate deterministic continuation check shown only if final review accepts it

### Active Task Without plannedDate

Expected while Question B remains open:

- not classified as overdue
- not silently included in R2
- treatment depends on final temporal-checkpoint decision

---

## 19. Review Resolution So Far

Claude findings and current resolutions:

```txt
F1 — Blocking: clarification questions could reintroduce forbidden inference
F2 — Important: absolute R5 threshold missed small Projects
F3 — Important: undated Tasks could remain invisible
F4 — Important: R4 could consume calibration data
F5 — Minor: duplicate absence/evidence gates were unclear
F6 — Minor: bulk Drop lacked empty-parent disclosure
```

Resolved now:

- F1 partially resolved: causal, emotional, motivational, capacity, and future-prediction questions are forbidden; free text is excluded from rule matching. Goal intent confirmation remains a narrowly defined open question.
- F2 resolved: R5 includes a proportional branch with minimum two active and two affected Tasks.
- F3 converted into Open Question B because its smallest local fix exposed a broader temporal-model problem across Task, Project, and Goal.
- F4 resolved: R4 counts only post-calibration observations.
- F5 resolved: R4 consumes one canonical evidence-quality calculation.
- F6 resolved: bulk actions disclose when a parent will remain active without executable children.

Additional accepted review responses:

- R2 seven-day threshold is conceptually independent from Discussion 016 severity thresholds.
- exact recommendation counts are deferred while the qualitative limited/chunked contract remains.
- Project target-date extension is removed until a canonical Project target date is accepted.
- Task Split remains deferred for a later exact generation contract.

---

## 20. Stabilized Decisions

- AI may not perform free semantic interpretation.
- Reconcile intelligence uses only canonical data, events, and deterministic metrics.
- AI may not ask causal, emotional, motivational, capacity, or future-prediction questions.
- Free-text explanations do not feed rule matching or rationale inference.
- Recommendations require an accepted matched rule.
- Ownership is the first grouping boundary.
- Direct Goal-owned and standalone work are supported.
- Semantic similarity and duplicate detection are excluded from MVP.
- Protected items cannot receive unsafe destructive suggestions.
- Repeated Carry begins at two.
- Raw MISSED count does not establish Routine mismatch.
- R4 uses post-calibration observations and canonical evidence quality.
- R5 supports both absolute and proportional Project patterns.
- Vague Project review recommendations are forbidden.
- Normal Reconcile does not recommend Project terminal actions.
- Normal Reconcile does not infer Goal achievement or abandonment.
- Consequential actions require explicit user confirmation.
- Bulk removal must disclose empty-parent consequences.
- Reconcile may pass structured evidence to Adaptive Planning but does not rebuild the plan.
- Goal intent confirmation remains open.
- active-entity temporal checkpoint requirements remain open.

---

## 21. Remaining Open Product Questions

Only the following two model-level questions are intentionally prioritized for the next Claude review:

### Q1. Goal continuation confirmation

Should Reconcile support a deterministic, closed-form continuation check after meaningful absence and stale descendant execution, while continuing to forbid all causal, emotional, and predictive questions?

Current proposal: yes, under Section 16 constraints.

### Q2. Temporal checkpoint invariant

Should every active Task, Project, Goal, and Routine have a future temporal checkpoint so no active entity can become indefinitely invisible?

Current proposal: yes, through entity-specific dates or a derived `nextTemporalCheckpoint` abstraction, under Section 17.

Lower-level thresholds remain validation details, but these two questions may amend previously accepted product semantics and require explicit review.

---

## 22. Mind Map Impact — Record Only, Do Not Apply Yet

### Product Vision

Add:

- AI organizes recorded evidence but does not interview the user about causes or emotions.
- exact problems and limited actions replace vague advice.
- active commitments should not become temporally invisible.
- returning after absence may require explicit intent confirmation without judgment.

### MVP Core Loop

```txt
Open Reconcile
→ deterministic facts and ownership groups
→ approved metrics
→ bounded rules
→ exact evidence and allowed actions
→ user decision
→ execution history update
→ optional structured handoff to Adaptive Planning
```

Potential extension pending review:

```txt
meaningful absence + stale Goal descendants
→ deterministic Goal continuation check
```

### User Flow

Add:

```txt
simple item
→ deterministic quick actions

matched rule
→ exact problem
→ observed metrics
→ limited actions
→ confirmation

consequential bulk action
→ full preview
→ empty-parent warning when applicable
→ confirmation
```

Pending temporal proposal:

```txt
reviewDate reached
→ bounded placement or continuation checkpoint
```

### AI Responsibilities

Add:

- explain matched deterministic rules
- present only accepted recommendation types
- summarize recorded metrics
- consolidate repeated recommendations
- create structured session summary

Remove:

- targeted causal clarification
- free-text interpretation
- semantic duplicate detection
- personality, motivation, or lifestyle inference
- vague Project or Goal advice

### AI Guardrails

Add:

- no causal, emotional, motivational, capacity, or future-prediction questions
- no free-text input to rule matching
- no recommendation without a matched rule
- no invented metric or problem type
- no destructive action for protected items
- no raw MISSED-count interpretation
- no absence-as-failure inference
- no autonomous lifecycle change
- no vague recommendation without exact evidence

### Data Events

Candidates for Discussion 019:

```txt
RECONCILE_RULE_EVALUATED
RECONCILE_RULE_MATCHED
RECONCILE_RECOMMENDATION_PRESENTED
RECONCILE_RECOMMENDATION_ACCEPTED
RECONCILE_RECOMMENDATION_REJECTED
RECONCILE_BULK_PREVIEWED
RECONCILE_BULK_CONFIRMED
RECONCILE_EMPTY_PARENT_CONSEQUENCE_SHOWN
RECONCILE_ADAPTIVE_HANDOFF_CREATED
```

Pending review:

```txt
GOAL_CONTINUATION_CHECK_PRESENTED
GOAL_CONTINUATION_REAFFIRMED
TEMPORAL_CHECKPOINT_REACHED
TEMPORAL_CHECKPOINT_RESCHEDULED
```

### Traction Metrics

Valid metrics:

- recommendation acceptance and rejection by rule ID
- user edits before confirmation
- bulk-preview cancellation rate
- protected-item exclusion correctness
- repeated Carry after R1 resolution
- Routine change acceptance after R4
- return-to-execution after Reconcile

Guardrails:

- unsupported recommendation rate, expected zero
- recommendation without matched rule, expected zero
- causal/emotional question rate, expected zero
- free-text-derived metric rate, expected zero
- destructive suggestion involving protected item, expected zero
- absence-contaminated R4 recommendation rate, expected zero

### Current Decisions

Record all stabilized decisions from Section 20.

### Open Questions

Keep only the two priority model questions:

- deterministic Goal continuation confirmation
- mandatory temporal checkpoint for every active entity

Do not yet record either proposal as accepted architecture.

---

## 23. Affected Formal Documents — Record Only, Do Not Update Yet

Accepted corrections should later update or create:

- Reconcile deterministic fact specification
- bounded Reconcile rule catalog
- Reconcile recommendation contract
- prohibited-question and free-text-input policy
- protected-item policy
- ownership-first grouping specification
- bulk-action safety and empty-parent disclosure policy
- consequence-disclosure specification
- Reconcile-to-Adaptive-Planning evidence handoff
- data and event specification in Discussion 019
- runtime and rule-versioning specification in Discussion 020
- validation plan in Discussion 021
- implementation plan in Discussion 022

If Open Question A is accepted, affected documents also include:

- Goal continuation-check specification
- Goal lifecycle presentation policy
- Discussion 016 trigger integration

If Open Question B is accepted, explicit amendments are required for:

- Discussion 012 core entity invariants
- Discussion 014 PlanningDraft fields and validation
- Discussion 015 Today, overdue, and checkpoint semantics
- Discussion 016 eligibility and severity inputs
- Discussion 019 persistence and events

Potential ADRs:

- bounded rule-driven intelligence instead of free interpretation
- no causal or emotional Reconcile interview
- ownership-first grouping
- no autonomous consequential actions
- explicit active-entity temporal checkpoint model, if accepted

No Mind Map or formal document is updated before consolidation.

---

## 24. Structured Review Request for Claude

Review this exact revision for coherence, safety, MVP simplicity, and closure readiness.

For every issue, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected decision or rule
4. concrete failure scenario
5. why it conflicts or is incomplete
6. smallest coherent correction
7. affected earlier or later discussion
```

First verify the accepted corrections:

- whether all causal, emotional, motivational, capacity, and predictive questions are now forbidden clearly enough
- whether free text is fully prevented from entering rule matching
- whether the R5 proportional branch handles small Projects without false escalation
- whether R4 correctly excludes calibration observations
- whether evidence quality is canonical rather than duplicated
- whether bulk Drop and Backlog actions disclose empty-parent consequences
- whether removal of `EXTEND_PROJECT_TARGET_DATE` is consistent with Discussion 014

Then focus deeply on the two remaining open questions:

### Review Question A — Goal continuation confirmation

- Is a closed-form `CONTINUE_GOAL | ABANDON_GOAL | REVIEW_LATER` check a legitimate deterministic lifecycle confirmation or an unacceptable Reconcile interview?
- Is absence plus stale descendant execution a sufficient trigger?
- Should abandonment navigate to a separate terminal flow rather than appear as a direct option?
- What suppression and evidence boundaries are required?
- Does this conflict with Discussion 012, 015, or 016?

### Review Question B — Temporal checkpoints

- Is `No ACTIVE entity may be temporally invisible` a coherent invariant?
- Should review dates be conditional or mandatory?
- Should Goal and Project have target dates, review dates, or both?
- Should Task without `plannedDate` require `reviewDate`?
- Is `nextTemporalCheckpoint` canonical or derived?
- How should review-checkpoint expiry differ from overdue execution?
- Does this require reopening accepted Discussions 012, 014, and 015 through an amendment?
- Does it remove the need for a permanent undated backlog without over-constraining legitimate work?

Do not expand into database schemas, API payloads, provider choice, UI styling, full Adaptive Planning thresholds, analytics implementation, or implementation sequencing.

This discussion remains open until Claude reviews this exact revision, Mahdi resolves the two remaining model questions, and final Review Resolution, Mind Map Impact, and Affected Formal Documents are recorded.
