# Discussion 015 — Task and Routine Execution Model

## Status

Open for GPT × Claude review.

This version contains GPT's proposed execution decisions. It must remain open until Claude reviews this exact revision, Mahdi resolves product choices, accepted corrections are integrated, and final resolution, Mind Map Impact, and Affected Formal Documents are recorded.

Related accepted discussions:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/014-ai-planning-output-contract]]

---

## 1. Scope

This discussion defines how approved Tasks, Projects, Routines, RoutineOccurrences, and the Today view behave during normal execution.

It answers:

```txt
How is Today assembled?
When does a Task appear in Today?
How are RoutineOccurrences created and resolved?
What happens when scheduled work is not completed?
How do Task, Project, Routine, and occurrence lifecycle transitions behave?
How do Project completion and stopping affect child work?
What may be corrected retroactively?
How are local dates and timezone changes handled?
Which unresolved execution facts become eligible for Reconcile?
```

This discussion consumes the canonical concepts and ownership rules accepted in Discussion 012. It does not redefine them.

---

## 2. Explicitly Out of Scope

This discussion does not define:

- AI planning conversation — Discussion 013
- PlanningDraft structure and approval — Discussion 014
- Reconcile grouping intelligence, adaptation suggestions, or signal thresholds — Discussions 016–017
- safety, privacy, and provider-failure behavior — Discussion 018
- persistence tables, transactions, indexes, or event schema — Discussion 019
- runtime/API/provider architecture — Discussion 020
- final validation strategy — Discussion 021
- implementation sequencing — Discussion 022
- exact Today visual design, animations, color, spacing, or responsive components

It defines product-level execution semantics that later technical and UX specifications must preserve.

---

## 3. Accepted Dependencies from Discussion 012

The canonical concepts are:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

Accepted ownership rules:

```txt
Project → one Goal or standalone
Task → one Goal, one Project, or standalone
Routine → one Goal, one Project, or standalone
RoutineOccurrence → exactly one Routine
```

Task and Routine may not belong to Goal and Project simultaneously.

Accepted lifecycle boundaries:

```txt
Goal: ACTIVE → ACHIEVED | ABANDONED
Project: ACTIVE → COMPLETED | STOPPED
Task: ACTIVE → COMPLETED | DROPPED
Routine: ACTIVE → STOPPED
RoutineOccurrence: PENDING → DONE | MISSED
```

Accepted invariants:

- direct Goal-owned Tasks and Routines are valid
- Project-owned Routines are specific to that Project
- Project completion automatically stops active Project-owned Routines
- historical RoutineOccurrences remain historical facts
- Carry changes Task placement; it is not a Task status
- RoutineOccurrence is never Carried
- execution facts do not prove Goal achievement

The earlier skeleton statement that Task and Routine never link directly to Goal was incorrect and is removed because it contradicted accepted Discussion 012.

---

## 4. Core Execution Principles

### 4.1 Today Is a Local-Date Execution View

Today is not a persisted Plan entity and does not own Tasks or Routines.

It is a view assembled for one user-local calendar date from canonical execution facts.

```txt
Today(localDate)
= active Tasks placed on localDate
+ pending RoutineOccurrences scheduled for localDate
+ unresolved earlier Tasks explicitly carried to localDate
+ eligible execution feedback or Reconcile entry points
```

Today must not silently invent work, duplicate entities, or convert missed Routine occurrences into new debt.

### 4.2 Task and Routine Execute Differently

A Task is one piece of work.

```txt
Task definition
= the executable item itself
```

A Routine is a recurring definition.

```txt
Routine definition
→ scheduled RoutineOccurrence
→ occurrence execution result
```

Completing one RoutineOccurrence does not complete or stop the Routine.

### 4.3 Local Date Is Authoritative for Daily Execution

Today membership is determined by user-local date, not by raw UTC date.

UTC may be used for storage and event timestamps, but product meaning is based on the user's effective timezone and local calendar day.

### 4.4 No Hidden Carry

Unfinished Tasks do not silently move forward merely because midnight passed.

A Task stays active but unresolved until the user or an accepted Reconcile action explicitly:

```txt
carry it to another date
complete it
drop it
leave it unresolved for later review
```

RoutineOccurrences never carry.

### 4.5 Historical Facts Are Preserved

Past execution results must not be rewritten merely because a parent later completes, stops, or changes.

Project and Routine lifecycle changes affect current and future execution eligibility, not historical occurrence facts.

---

## 5. Today Assembly Rules

### 5.1 Eligible Task Sources

Today may contain active Tasks from any accepted ownership form:

```txt
standalone Task
Goal-owned Task
Project-owned Task
```

Ownership affects context and lifecycle consequences but not basic Today eligibility.

### 5.2 When a Task Enters Today

An active Task appears in Today when one of these conditions is true:

1. its authoritative `plannedDate` equals the current local date;
2. it was explicitly carried to the current local date;
3. the user manually places an undated active Task into Today;
4. an accepted Reconcile action assigns it to the current local date.

An undated Task does not automatically appear every day.

A future Task does not appear early merely because it belongs to an active Goal or Project.

A completed or dropped Task never appears as executable work in Today, though it may remain visible through history or parent context.

### 5.3 Task Ordering

Discussion 012 forbids a generic persisted canonical Task order.

Today may use a temporary presentation order based on product rules such as:

- explicit time, if supported
- user-local placement
- manually chosen Today order
- AI suggestion
- stable creation fallback

This ordering is view-level or date-level execution metadata, not a universal Task hierarchy field.

Exact persistence belongs to Discussion 019.

### 5.4 Eligible Routine Sources

Today may contain RoutineOccurrences from:

```txt
standalone Routine
Goal-owned Routine
Project-owned Routine
```

The user executes the occurrence, not the Routine definition itself.

### 5.5 Today Does Not Show Future Occurrences as Due

Future RoutineOccurrences may appear in a calendar or preview, but Today shows only the occurrence for the current local date as actionable.

Past unresolved occurrences belong to history or Reconcile, not to today's active checklist.

---

## 6. Task Execution Model

### 6.1 Task Lifecycle

```txt
ACTIVE → COMPLETED
ACTIVE → DROPPED
```

`ACTIVE` includes dated, undated, future, due-today, and unresolved-past Tasks.

These are placement conditions, not additional canonical lifecycle states.

### 6.2 Complete Task

Completing a Task:

- changes Task lifecycle to `COMPLETED`
- records the completion timestamp and effective local date
- removes it from active Today execution
- preserves its parent relationship and history
- does not automatically complete its Project or achieve its Goal

### 6.3 Drop Task

Dropping a Task means the user no longer intends to perform that action.

It:

- changes lifecycle to `DROPPED`
- removes future and current execution placement
- preserves history and parent relationship
- may become a Reconcile signal for the parent
- does not automatically stop sibling work

### 6.4 Carry Task

Carry is an explicit placement transition:

```txt
Task remains ACTIVE
old plannedDate → new plannedDate
carry event recorded
```

Rules:

- only an active Task may be carried
- the destination local date must be explicit
- Carry does not create a duplicate Task
- Carry preserves ownership and identity
- repeated Carry may become a Reconcile signal
- Carry history should remain available even though only the current planned date is authoritative

### 6.5 Leave Task Unresolved

When a due date passes without completion, drop, or Carry:

- the Task remains `ACTIVE`
- its last planned date remains historical execution context
- it does not silently receive today's date
- it becomes eligible for Reconcile
- it may be shown in an overdue/unresolved section when the user opens the product

`OVERDUE` is a derived condition, not a canonical Task status.

### 6.6 Retroactive Task Completion

The user may complete an active past-due Task retroactively.

The product should distinguish:

```txt
recordedAt = when the correction was entered
completedForLocalDate = when the user says the work occurred
```

For MVP, retroactive completion should be allowed only within a bounded correction window proposed here as seven local calendar days.

This limit reduces accidental history rewriting while allowing ordinary correction after offline use or forgetfulness.

Claude should review whether seven days is the right product boundary or whether the window should remain deferred.

### 6.7 Reopen Completed or Dropped Tasks

MVP proposal:

- completed Task may be corrected back to `ACTIVE` within the same bounded correction window
- dropped Task may be restored to `ACTIVE` through an explicit action
- restoration requires a new explicit planned date or returns the Task to an undated active state
- reopening must not erase the original completion/drop event

This is correction, not silent history replacement.

---

## 7. Routine Execution Model

### 7.1 Routine Lifecycle

```txt
ACTIVE → STOPPED
```

MVP does not introduce a canonical `PAUSED` status.

A user who temporarily does not want future occurrences may stop the Routine and later explicitly reactivate it as a new active execution phase while preserving the same Routine identity and history.

Alternative terminology such as Resume is a UX choice; the product-level effect must create a new effective recurrence phase rather than rewriting past expectations.

Claude should specifically review whether reactivation without a canonical PAUSED state is coherent or whether MVP should simply require creating a new Routine.

### 7.2 Stop Routine

Stopping a Routine:

- prevents generation or eligibility of occurrences after the effective stop boundary
- does not change historical occurrences
- does not mark pending past occurrences Done
- resolves same-day and known-future occurrences according to Section 9

### 7.3 Edit Routine Metadata

Editing title, description, preferred time, duration, or recurrence affects future execution only unless the user explicitly corrects historical data.

Past RoutineOccurrences keep the historical meaning necessary to interpret them.

### 7.4 Recurrence Edit Effective Date

A recurrence edit requires an explicit effective local date.

Default proposal:

```txt
effectiveDate = next local date after the edit
```

The user may choose the current local date only when today's occurrence is still `PENDING`.

Rules:

- past occurrences are never regenerated from the new rule
- future occurrence eligibility is recomputed from the effective date
- a DONE or MISSED occurrence for today is not replaced by changing recurrence
- a same-day PENDING occurrence may be added or removed only through explicit confirmation when today's applicability changes

### 7.5 Goal-Owned and Standalone Routine Independence

Completing or stopping a Project does not affect Goal-owned or standalone Routines.

A Routine should be Project-owned only when it is specific to that finite Project.

---

## 8. RoutineOccurrence Generation Model

### 8.1 Canonical Meaning

A RoutineOccurrence represents one Routine scheduled on one user-local date.

Conceptual uniqueness:

```txt
one Routine + one scheduled local date
→ at most one ordinary RoutineOccurrence
```

If future product requirements allow multiple daily repetitions, that requires an explicit model extension and is not silently represented by duplicate occurrences.

### 8.2 Proposed Generation Strategy

Product semantics should not depend on whether future occurrences are physically pre-created or lazily materialized.

The required behavior is:

- each scheduled local date has a stable occurrence identity once it becomes execution-relevant
- opening the app must materialize all due occurrence facts needed for Today and missed-history resolution
- duplicate occurrences for the same Routine and scheduled local date are forbidden
- technical generation horizon belongs to Discussion 019

Recommended MVP behavior:

```txt
materialize current local date when Today is assembled
materialize missing past dates since the last evaluated date in bounded batches
optionally materialize a short future preview window
```

The system must derive the same user-visible result whether the app was opened every day or not.

### 8.3 Opening After Several Days

When the app is opened after one or more scheduled dates were not evaluated:

1. determine the user's effective timezone history where available;
2. enumerate scheduled local dates since the Routine's last evaluated boundary;
3. create or derive missing occurrences idempotently;
4. mark past unresolved scheduled occurrences `MISSED`;
5. create or preserve the current-date occurrence as `PENDING` when applicable;
6. do not carry past occurrences into Today;
7. expose the execution pattern to Reconcile when relevant.

This catch-up process must be bounded operationally, but may not silently erase recent missed facts.

Long inactivity compaction policy is deferred to Discussion 019, provided product truth remains honest.

### 8.4 Occurrence Lifecycle

```txt
PENDING → DONE
PENDING → MISSED
```

MVP does not add `SKIPPED` as a canonical state.

Reason:

- `DONE` means the behavior occurred
- `MISSED` means the scheduled behavior was not recorded as done by the resolution boundary
- explanations such as intentional rest, illness, or not-applicable may be optional execution feedback, not another lifecycle state

Claude should review whether merging intentional skip into MISSED creates misleading history.

### 8.5 When PENDING Becomes MISSED

A pending occurrence becomes `MISSED` after its scheduled local date ends and the next product evaluation occurs.

The system does not require a midnight background job for semantic correctness.

```txt
scheduled local day ended
+ next evaluation occurs
→ unresolved PENDING becomes MISSED
```

The recorded transition timestamp may be later than the scheduled date, but the missed fact belongs to the scheduled local date.

### 8.6 Complete Current Occurrence

Marking today's pending occurrence Done:

- changes it to `DONE`
- records completion timestamp
- removes it from active Today work
- leaves the Routine active
- does not generate bonus credit or streak guarantees

### 8.7 Retroactive Occurrence Correction

The user may correct a recent past occurrence:

```txt
MISSED → DONE
DONE → MISSED
```

Proposal:

- correction allowed within seven local calendar days
- correction records who/what changed it and when
- scheduled local date never changes
- a corrected occurrence is never Carried
- corrections may update Reconcile evidence

Claude should review the seven-day correction window together with Task correction rules.

### 8.8 No Carry for RoutineOccurrence

A missed occurrence remains attached to its original local date.

If the user wants to perform the behavior today:

- today's scheduled occurrence may be completed when one exists;
- or the user may perform an ad-hoc action outside the Routine occurrence model;
- the missed occurrence itself is not moved or duplicated.

MVP does not create replacement debt occurrences.

---

## 9. Project Completion and Stop Cascades

### 9.1 Project Completion Preconditions

A Project may be marked `COMPLETED` only through explicit user action.

MVP proposal:

- unresolved active child Tasks do not hard-block Project completion
- before confirmation, the product must show all active child Tasks and require one resolution choice

Valid choices:

```txt
1. complete Project and drop all unresolved child Tasks
2. complete Project and explicitly detach eligible Tasks to Goal or standalone
3. cancel Project completion and resolve Tasks first
```

The system must not leave active Tasks attached to a completed Project.

Project-owned active Routines are automatically stopped and cannot be detached implicitly during completion.

A user who wants a Routine to survive must explicitly reclassify its ownership before completing the Project.

### 9.2 Project Stop Preconditions

Stopping a Project means the finite effort ends without completion.

Before confirmation, unresolved child Tasks must be handled explicitly:

```txt
1. stop Project and drop unresolved child Tasks
2. stop Project and explicitly detach eligible Tasks
3. cancel stop
```

All active Project-owned Routines stop automatically.

### 9.3 Cascade Boundary

The Project lifecycle transition and automatic Routine stops share one effective product operation.

```txt
Project ACTIVE → COMPLETED | STOPPED
→ active Project-owned Routines → STOPPED
```

The cascade must be atomic from the user's perspective. Technical transaction behavior belongs to Discussion 019.

### 9.4 Same-Day RoutineOccurrence Boundary

When a Project is completed or stopped on local date D:

- occurrences dated before D remain historical
- future occurrences after D are cancelled from execution eligibility
- a same-day `DONE` or `MISSED` occurrence remains unchanged
- a same-day `PENDING` occurrence requires explicit treatment

Proposed same-day rule:

```txt
if scheduled time has passed or user has interacted with it:
    keep occurrence PENDING until user resolves it or day ends
else:
    remove/cancel today's pending execution expectation
```

However, `CANCELLED` is not an accepted occurrence lifecycle state in Discussion 012.

Therefore the cleaner MVP proposal is:

- if today's occurrence already exists, keep it as a historical scheduled fact;
- if still PENDING when the day ends, it becomes MISSED;
- stopping the Project prevents only dates after D;
- UI explains that today's occurrence belonged to the Project before it stopped.

Claude should review whether this is semantically honest or whether the stop operation needs an occurrence-level cancellation concept in a later revision.

### 9.5 Known Future Occurrences

After a Project-owned Routine stops:

- future occurrence records, if physically pre-created, must no longer appear as executable work
- they may be deleted if they were merely technical projections with no user-visible interaction
- they must not be rewritten when they already contain user interaction or historical meaning

The product contract treats them as non-existent future expectations after the stop effective boundary.

Exact storage cleanup belongs to Discussion 019.

### 9.6 Reopen Project

MVP proposal:

- completed or stopped Project is not directly reopened
- the user may duplicate or restart it through an explicit new active phase later
- historical completion/stop remains intact

This avoids silently reversing cascaded Routine shutdown and resolved child Tasks.

Claude should review whether first-version correction requires a bounded undo action immediately after transition.

---

## 10. Goal Lifecycle Effects on Execution

### 10.1 Achieve Goal

Achieving a Goal is explicit user confirmation of real-world outcome.

Before confirmation, the product must show active direct child Tasks, active direct Routines, and active Projects.

Proposed rule:

- achieving Goal does not silently complete Projects or Tasks
- user must explicitly resolve active direct child work
- active child Projects must be completed, stopped, or detached before Goal achievement is finalized
- active direct Tasks must be completed, dropped, or detached
- active direct Routines must be stopped or detached

No active execution entity remains owned by an achieved Goal.

### 10.2 Abandon Goal

Abandoning a Goal similarly requires explicit child resolution.

Default cascade proposal:

- direct Tasks are dropped unless explicitly detached
- direct Routines are stopped unless explicitly detached
- child Projects are stopped unless explicitly detached as standalone

This mirrors the child-obedience principle accepted for PlanningDraft review in Discussion 014.

Detailed Goal lifecycle UX may later receive its own specification, but execution consistency must not allow active children under inactive Goals.

---

## 11. Timezone and Local-Date Rules

### 11.1 Effective Timezone

Each execution fact uses the user's effective IANA timezone, not a fixed numeric UTC offset.

Examples:

```txt
Europe/Berlin
Asia/Tehran
America/Toronto
```

This allows daylight-saving transitions to resolve correctly.

### 11.2 Scheduled Local Date Stability

Once a Task placement or RoutineOccurrence is associated with a scheduled local date, timezone changes must not silently rewrite its historical date.

Store or preserve enough context to interpret:

- scheduled local date
- effective timezone used for that schedule
- UTC event timestamps

Exact fields belong to Discussion 019.

### 11.3 User Travels or Changes Timezone

Prospective behavior:

- new Today views use the newly effective timezone
- future Routine schedule evaluation uses the new timezone unless the Routine is explicitly anchored to another timezone
- past occurrences retain original scheduled local dates and timezone context
- existing current-day ambiguity must not create duplicate occurrences

MVP proposal excludes per-Routine fixed timezones unless a concrete use case requires them.

### 11.4 Daylight-Saving Boundaries

Date-based recurrence remains attached to local calendar dates.

If preferred time falls into a nonexistent or repeated local clock time, implementation must choose a deterministic valid instant while preserving the intended local date.

Exact resolution strategy belongs to Discussions 019–020.

---

## 12. Reconcile Eligibility

This discussion identifies execution facts eligible for Reconcile but does not define grouping or suggestions.

### 12.1 Task Signals

An active Task becomes Reconcile-eligible when:

- its planned local date passed unresolved
- it has been carried repeatedly
- it was dropped while supporting an active parent
- it was reopened or corrected
- its parent is being completed, stopped, achieved, or abandoned with unresolved ownership

### 12.2 Routine Signals

A Routine or its execution pattern becomes Reconcile-eligible when:

- recent occurrences are repeatedly missed
- occurrence corrections materially change the observed pattern
- recurrence is repeatedly edited
- a parent lifecycle change stops it
- the first calibration week ends
- signal-based continuation detects possible overload or poor fit

### 12.3 Project and Goal Signals

A Project or Goal becomes Reconcile-eligible when:

- child work is repeatedly unresolved, carried, dropped, or missed
- the user attempts a lifecycle transition with active children
- planned execution and actual execution meaningfully diverge

Eligibility does not authorize automatic semantic changes. Reconcile proposals require later accepted rules.

---

## 13. Direct Answers to Original Questions

### 1. How is Today assembled?

From active Tasks placed on the current local date and pending RoutineOccurrences scheduled for that date, across standalone, Goal-owned, and Project-owned work.

### 2. When does a Task enter Today?

When its authoritative planned date equals today, it is explicitly carried or placed into today, or an accepted Reconcile action assigns it there.

### 3. When and how are RoutineOccurrences generated?

They are idempotently materialized or derived for scheduled local dates. User-visible behavior must be identical whether generated lazily or ahead of time.

### 4. What happens if the app is not opened?

On next evaluation, missing scheduled dates are reconstructed; past unresolved occurrences become MISSED and are not carried.

### 5. Which transitions exist?

```txt
Task: ACTIVE → COMPLETED | DROPPED
Project: ACTIVE → COMPLETED | STOPPED
Routine: ACTIVE → STOPPED
Occurrence: PENDING → DONE | MISSED
```

Carry and correction are transitions/events, not extra lifecycle statuses.

### 6. Done, Missed, or Skipped?

MVP proposal uses DONE and MISSED only. Intentional explanation may be feedback metadata rather than SKIPPED status.

### 7. Can an occurrence be Carried?

No.

### 8. How do stop and edit work for Routine?

Stop ends future eligibility. Edits apply prospectively from an explicit effective local date. Canonical pause is excluded from MVP.

### 9. Does recurrence editing affect future occurrences only?

Yes. Default effective date is tomorrow; today may be selected only while today's occurrence is still pending and the consequence is explicit.

### 10. Can users correct past execution?

Yes, within a proposed bounded seven-day correction window, with audit history preserved.

### 11. How are timezone boundaries handled?

By IANA timezone and scheduled local-date semantics, while preserving historical timezone context.

### 12. What becomes eligible for Reconcile?

Unresolved past Tasks, repeated Carry, missed patterns, corrections, recurrence churn, calibration transitions, and parent lifecycle conflicts.

### 13. When do Project Routines stop after Project completion?

As part of the same effective product operation.

### 14. Does Project STOPPED stop active child Routines?

Yes.

### 15. What happens to same-day occurrences?

Existing same-day occurrences remain historical scheduled facts. The stop boundary prevents dates after the effective local date.

### 16. What happens to known future occurrences?

They lose execution eligibility and should not appear. Pure future projections may be removed technically.

### 17. What happens to active child Tasks?

They must be explicitly dropped, detached/reparented, or resolved before Project completion/stop is finalized. No active Task remains attached to an inactive Project.

### 18. Can terminal entities be reopened?

Proposal: bounded Task/occurrence correction is allowed; Project terminal states are not reopened in MVP except possible immediate undo to be reviewed.

### 19. Must Project completion be blocked by active Tasks?

It is blocked only until the user explicitly chooses how unresolved child Tasks are resolved. Tasks need not all be completed.

---

## 14. Scenario Checks

### Scenario A — Missed App Opens

```txt
Routine: Practice English every weekday
App last opened Monday
App next opened Thursday
```

Expected:

- Tuesday and Wednesday scheduled occurrences exist or are derived
- both become MISSED if not recorded Done
- Thursday occurrence is PENDING
- Tuesday and Wednesday do not appear as debt in Thursday Today

### Scenario B — Carry Task

```txt
Task: Submit application
plannedDate: Tuesday
```

User carries to Thursday.

Expected:

- same Task remains ACTIVE
- authoritative plannedDate becomes Thursday
- Tuesday placement remains in Carry history
- no duplicate Task is created

### Scenario C — Complete Project with Active Work

```txt
Project: Build portfolio
Task: Write case study — ACTIVE
Routine: Work on portfolio weekdays — ACTIVE
```

Expected:

- completion action shows unresolved Task
- user drops or explicitly detaches the Task, or cancels completion
- Project-owned Routine stops automatically
- no active child remains attached after completion

### Scenario D — Stop Project on Occurrence Date

A Project-owned Routine has a pending occurrence today and the Project is stopped today.

Expected proposal:

- today's occurrence remains tied to today's historical schedule
- it may still be completed today
- if unresolved after today, it becomes MISSED
- no occurrence is eligible after today

### Scenario E — Recurrence Edit

```txt
Routine: Monday, Wednesday, Friday
Edit on Wednesday to Tuesday, Thursday
```

Expected default:

- past Monday remains unchanged
- today's existing Wednesday occurrence remains unchanged unless user explicitly applies edit today while it is pending
- new rule applies from Thursday by default
- no historical regeneration

### Scenario F — Travel Across Timezones

User travels from Tehran to Berlin.

Expected:

- past occurrences retain Tehran local-date context
- future Today assembly uses Berlin timezone
- no duplicate occurrence is created for the transition date

### Scenario G — Goal Abandonment

Goal has active direct Task, Routine, and Project.

Expected:

- user sees all dependent active work
- default cascade drops Task, stops Routine, and stops Project
- explicit detachment may preserve eligible children
- nothing remains actively owned by abandoned Goal

---

## 15. Included, Excluded, and Deferred

### Included Now

- Today membership semantics
- Task execution and Carry
- Routine lifecycle and prospective editing
- RoutineOccurrence generation-independent behavior
- missed-date catch-up
- retroactive correction proposal
- Project completion/stop cascades
- active child resolution
- Goal terminal lifecycle execution consistency
- local-date and timezone semantics
- Reconcile eligibility signals

### Explicitly Excluded

- persisted Plan entity
- automatic hidden Carry
- RoutineOccurrence Carry
- Task dependency graph
- universal persisted Task order
- automatic Goal achievement
- silent child detachment or reparenting
- silent history rewriting
- mandatory streaks or debt accumulation

### Deferred

- exact Today UI
- exact correction-window duration if Claude or Mahdi rejects seven days
- whether intentional skip needs canonical `SKIPPED`
- whether Project terminal actions support immediate undo
- whether stopped Routine may reactivate with same identity or requires a new Routine
- Reconcile grouping and action rules
- persistence and event implementation
- future occurrence materialization horizon
- long-inactivity compaction
- exact DST instant-resolution rules

---

## 16. Mind Map Impact — Record Only, Do Not Apply Yet

After Discussion 021 consolidation, add or revise:

### User Flow

```txt
Approved execution window
→ Today assembled by local date
→ complete / carry / drop Task
→ complete RoutineOccurrence
→ unresolved facts become history and/or Reconcile signals
```

### Product Model Execution Notes

- Today is a derived local-date view, not an entity
- Task Carry changes placement, not status
- overdue is derived, not a Task status
- Routine executes through RoutineOccurrence
- RoutineOccurrence never carries
- past pending occurrence becomes Missed on next evaluation
- Project completion/stop stops Project-owned Routines
- inactive parent cannot retain active dependent children
- historical execution facts remain preserved

### AI and Adaptive Behavior Inputs

- first week provides calibration evidence
- unresolved Tasks and repeated Carry feed Reconcile
- missed Routine patterns feed Reconcile
- continuation remains signal-based after first explicit transition

No Mind Map change is applied yet.

---

## 17. Affected Formal Documents — Record Only, Do Not Update Yet

After consolidation, accepted decisions should update or create:

- Today execution specification
- Task lifecycle and Carry specification
- Routine lifecycle and recurrence-edit specification
- RoutineOccurrence generation and correction specification
- Project and Goal cascade specification
- timezone and local-date specification
- Reconcile input-signal specification for Discussions 016–017
- data and event specification in Discussion 019
- API/runtime specification in Discussion 020
- validation plan in Discussion 021
- implementation plan in Discussion 022

Potential ADRs:

- Today as a derived local-date view rather than persisted Plan
- lazy/idempotent occurrence materialization semantics
- terminal parent cascade and child-resolution behavior
- historical correction model

No formal document is updated from this discussion alone before consolidation.

---

## 18. Structured Review Request for Claude

Review this exact revision according to `claude-review-standards.md`.

For every issue, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected decision
4. concrete failure scenario
5. why it conflicts or is incomplete
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- contradictions with accepted Discussion 012 or 014
- whether Today membership can be determined consistently
- whether overdue-as-derived rather than status is coherent
- whether explicit Carry avoids silent rescheduling
- whether lazy occurrence generation produces the same facts after app inactivity
- whether DONE/MISSED without SKIPPED is truthful enough
- whether seven-day retroactive correction is justified or arbitrary
- whether recurrence edit effective-date behavior is complete
- whether stopping or completing a Project handles same-day occurrences coherently
- whether active child Task resolution creates destructive or confusing cascades
- whether achieved/abandoned Goals may coherently retain no active owned work
- whether Routine reactivation without PAUSED is coherent
- whether any proposal silently redefines a core entity
- whether any rule cannot be validated deterministically

Do not expand into:

- database table design
- API payloads
- provider or prompt selection
- exact Reconcile scoring thresholds
- visual styling
- analytics
- implementation sequencing

This discussion remains open until Claude reviews this exact version and accepted corrections, final resolution, Mind Map impact, and affected formal documents are recorded.
