# Discussion 015 — Task and Routine Execution Model

## Status

Open for GPT × Claude review.

This revision records Mahdi's accepted execution decisions, separates Reconcile from Adaptive Planning, integrates Claude findings F1–F3, and keeps unresolved product choices explicit. It must remain open until Claude reviews this exact version and Mahdi accepts the final corrections.

Related accepted discussions:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/014-ai-planning-output-contract]]

---

## 1. Scope

This discussion defines how approved Tasks, Projects, Routines, RoutineOccurrences, and the Today view behave during normal execution.

It determines:

- how Today is assembled
- when Tasks become actionable today
- how RoutineOccurrences are derived or materialized
- how Task Carry, completion, dropping, and unresolved past work behave
- how Routine stop and recurrence editing behave
- how occurrence completion, missing, and historical correction behave
- how Project and Goal terminal transitions affect active children
- how local dates and timezones affect execution
- which execution facts become eligible for Reconcile

This discussion defines execution semantics only. It does not design the full Reconcile flow or the Adaptive Planning algorithm.

---

## 2. Explicitly Out of Scope

This discussion does not define:

- AI planning conversation — Discussion 013
- PlanningDraft structure and approval — Discussion 014
- Reconcile grouping, prioritization, or resolution suggestions — Discussions 016–017
- Adaptive Planning scoring, workload adjustment, or pattern thresholds
- safety and provider-failure behavior — Discussion 018
- persistence tables, transactions, and event schema — Discussion 019
- runtime, API, and provider architecture — Discussion 020
- final validation plan — Discussion 021
- implementation sequence — Discussion 022
- exact Today visual design

Later discussions may consume execution data defined here, but may not silently redefine these lifecycle semantics.

---

## 3. Accepted Dependencies from Discussion 012

Canonical concepts:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

Accepted ownership:

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
- Carry changes Task placement and is not a Task status
- RoutineOccurrence is never Carried
- execution facts do not prove Goal achievement

---

## 4. Separation of Execution, Reconcile, and Adaptive Planning

These are related but distinct responsibilities.

### 4.1 Execution

Execution records what happened to canonical work:

```txt
Task completed, carried, dropped, or left unresolved
RoutineOccurrence done or missed
Project completed or stopped
Routine stopped
```

### 4.2 Reconcile

Reconcile resolves unresolved or inconsistent past execution facts.

Examples:

- a Task date passed while the Task remained active
- a Task was carried repeatedly
- several old Tasks require explicit decisions
- missed or corrected RoutineOccurrences need review
- a parent lifecycle action conflicts with active children

Reconcile may update execution facts through explicit accepted actions. It does not independently design the next long-term or weekly plan.

### 4.3 Adaptive Planning

Adaptive Planning proposes future workload or schedule changes using reliable execution evidence.

It may use results produced through Reconcile, such as repeated Carry or Drop decisions, but it must not treat all missing data as poor performance.

Core guardrails:

```txt
absence is not failure
missing data is not negative evidence
one weak period is not a stable pattern
major adaptation requires repeated reliable evidence or explicit user feedback
```

A long user absence must reduce confidence rather than independently justify a major workload reduction.

The exact evidence model and pattern thresholds are outside Discussion 015.

---

## 5. Today Execution View

### 5.1 Today Is Derived

Today is not a persisted Plan entity.

It is a local-date execution view assembled from canonical Tasks and RoutineOccurrences.

```txt
Today(localDate)
= active Tasks whose authoritative plannedDate is localDate
+ pending RoutineOccurrences scheduled for localDate
+ relevant entry points for unresolved past work
```

Goal and Project may provide hierarchy and context but are not executable checklist items themselves.

### 5.2 Eligible Task Ownership

Today may contain:

- standalone Tasks
- Goal-owned Tasks
- Project-owned Tasks

Ownership does not change basic Today eligibility.

### 5.3 Proposed Task Entry Rule

Current proposal:

> A Task becomes actionable in Today only when its authoritative `plannedDate` equals the current local date.

Manual placement, Carry, AI-approved scheduling, or an accepted Reconcile action must all ultimately update the same authoritative `plannedDate`. They must not create a second hidden Today-membership mechanism.

Consequences:

- undated Task does not automatically appear every day
- future Task does not appear early
- completed or dropped Task is not executable in Today
- carrying a Task to today changes its `plannedDate` to today

Claude should review whether any valid Today-entry case requires separate date-level metadata rather than `plannedDate`.

### 5.4 Overdue Work

An overdue Task is:

```txt
Task.status = ACTIVE
and Task.plannedDate < currentLocalDate
```

`OVERDUE` is a derived condition, not a canonical status.

Current proposal:

- Today remains focused on work planned for today
- unresolved earlier Tasks appear in a separate unresolved/Reconcile entry area
- old Tasks do not silently remain inside the active Today checklist forever

Claude should review whether this separation creates any discoverability or execution contradiction.

### 5.5 Today Ordering

Discussion 012 forbids a generic persisted Task order.

Today may use temporary date-level or view-level ordering, but this must not become a universal canonical Task hierarchy field.

Exact persistence is deferred to Discussion 019.

---

## 6. Task Execution Model

### 6.1 Lifecycle

```txt
ACTIVE → COMPLETED
ACTIVE → DROPPED
```

Dated, undated, future, due-today, and unresolved-past are conditions of an active Task, not lifecycle states.

### 6.2 Complete

Completing a Task:

- changes it to `COMPLETED`
- records when completion was entered
- records the effective local date the user says it was completed
- removes it from active execution
- preserves ownership and history
- does not automatically complete its Project or achieve its Goal

### 6.3 Drop

Dropping a Task means the user no longer intends to perform it.

It:

- changes it to `DROPPED`
- removes active placement
- preserves history and parent relationship
- may become a Reconcile and Adaptive Planning signal
- does not automatically affect siblings

### 6.4 Carry

Carry is an explicit placement transition:

```txt
Task remains ACTIVE
old plannedDate → new plannedDate
Carry event is recorded
```

Rules:

- only an active Task may be carried
- destination local date must be explicit
- no duplicate Task is created
- identity and ownership remain unchanged
- current `plannedDate` is authoritative
- Carry history remains available
- no automatic midnight Carry exists

### 6.5 Unresolved Past Task

When a Task date passes without completion, Drop, or Carry:

- Task remains `ACTIVE`
- original planned date remains execution history
- Task does not silently receive today as its date
- Task becomes Reconcile-eligible
- Task appears through unresolved-history UX rather than as ordinary current-day work

### 6.6 Historical Task Correction

The user may correct historical execution facts.

The product should distinguish:

```txt
recordedAt = when the correction was entered
completedForLocalDate = when the user says the work occurred
```

Accepted correction principle:

- correction is not limited by a final fixed seven-day rule in this discussion
- long absence must not make truthful history permanently uncorrectable
- original events must not be silently erased
- older or consequential correction may require stronger confirmation and audit history

Open question:

> What product boundary, if any, limits historical Task correction?

Current proposal for Claude review:

- allow correction beyond seven days
- require confirmation for old or consequential edits
- preserve an audit trail
- defer exact retention and event implementation to Discussion 019

This boundary must not be confused with Adaptive Planning's weekly calibration window.

### 6.7 Reopen Completed or Dropped Task

Current proposal:

- a completed or dropped Task may be explicitly restored to `ACTIVE` as a correction
- restoration must preserve the original terminal event in history
- restored Task requires a new explicit date or becomes undated
- restoration is not silent history replacement

Claude should review whether this correction exception conflicts with the one-way lifecycle language in Discussion 012 or whether it must instead create a new continuation Task.

---

## 7. Routine Execution Model

### 7.1 Lifecycle

```txt
ACTIVE → STOPPED
```

MVP does not introduce `PAUSED`.

A stopped Routine does not transition back to `ACTIVE`.

### 7.2 Resume Semantics

Accepted correction from Claude F1:

> Resuming a stopped Routine creates a new active Routine using the stopped Routine as editable source material. It does not reactivate the same entity.

The previous Routine and its RoutineOccurrences remain unchanged.

The UI may label the action `Resume`, but its product meaning is:

```txt
stopped Routine remains STOPPED
→ create new ACTIVE continuation Routine
```

An optional conceptual `continues from` relationship may support history continuity, but its technical representation belongs to Discussion 019.

This follows the same one-way lifecycle principle proposed for terminal Projects.

### 7.3 Stop Routine

Stopping a Routine:

- prevents occurrence eligibility after the effective stop boundary
- preserves historical occurrences
- does not resolve past pending facts as Done
- does not carry missed behavior forward

### 7.4 Metadata and Recurrence Editing

Edits affect future execution only unless the user explicitly performs a historical correction.

A recurrence edit requires an effective local date.

Current proposal:

```txt
default effectiveDate = next local date after edit
```

The user may apply the edit today only if today's occurrence remains `PENDING` and the consequence is explicitly shown.

Rules:

- past occurrences are not regenerated
- DONE or MISSED occurrence for today is not replaced
- future eligibility is recomputed from the effective date
- same-day applicability changes require explicit confirmation

---

## 8. RoutineOccurrence Execution Model

### 8.1 Meaning and Uniqueness

A RoutineOccurrence represents one Routine scheduled for one user-local date.

```txt
one Routine + one scheduled local date
→ at most one ordinary RoutineOccurrence
```

Multiple repetitions per day require a future explicit model extension.

### 8.2 Derivation and Materialization Semantics

User-visible behavior must not depend on whether occurrences are pre-created or lazily materialized.

Required semantics:

- current-date occurrence becomes execution-relevant when Today is assembled
- missed dates since the last evaluated boundary can be reconstructed deterministically
- duplicate occurrences are forbidden
- future projections are not historical facts merely because they were technically pre-created
- exact generation horizon belongs to Discussion 019

Current proposal:

```txt
materialize or derive current local date
reconstruct missing past scheduled dates in bounded batches
optionally derive a short future preview
```

### 8.3 App Inactivity

When the app was not opened for scheduled dates:

- scheduled dates are reconstructed from recurrence
- unresolved past occurrences may become `MISSED` on next evaluation
- they do not appear as Carry debt in today's checklist
- user may later correct truthful historical execution

Important distinction:

```txt
MISSED occurrence fact
≠ proof that the user had low capacity
```

A long absence produces low-confidence Adaptive Planning evidence unless the user confirms what happened.

### 8.4 Lifecycle

```txt
PENDING → DONE
PENDING → MISSED
```

MVP does not add canonical `SKIPPED`.

- `DONE` means the behavior occurred
- `MISSED` means it was not recorded as Done by the resolution boundary
- reasons such as illness, intentional rest, or not-applicable may be optional feedback rather than another state

Accepted correction from Claude F3:

> `MISSED` is a neutral scheduled-execution fact, not a moral judgment, punishment, streak failure, or accumulated debt.

The UX must not imply shame merely from `MISSED`.

### 8.5 Resolution Boundary

A pending occurrence becomes `MISSED` after its scheduled local date ends and the product next evaluates it.

A midnight background job is not required for semantic correctness.

### 8.6 Historical Occurrence Correction

The user may explicitly correct:

```txt
MISSED → DONE
DONE → MISSED
```

Accepted principles:

- scheduled local date never changes
- correction records when it was entered
- original history is auditable
- occurrence is never Carried
- no final fixed seven-day limit is accepted here

Open question:

> Should historical occurrence correction remain unlimited with audit, require stronger confirmation after a threshold, or use another execution-policy boundary?

This is an execution-history question, not an Adaptive Planning calibration rule.

### 8.7 No Carry

A missed occurrence remains attached to its original date.

Performing the behavior today does not move or duplicate the missed occurrence.

---

## 9. Project Completion and Stop Cascades

### 9.1 Active Child Tasks

No active Task may remain owned by a terminal Project.

Current proposal:

Before `COMPLETED` or `STOPPED` is finalized, show unresolved active child Tasks and require one explicit resolution:

```txt
1. terminal transition + drop unresolved Tasks
2. terminal transition + explicitly detach/reparent eligible Tasks
3. cancel transition and resolve Tasks first
```

The system must not silently drop, preserve, or detach Tasks.

Claude should review whether this resolution interaction is sufficiently safe and whether completing a Project should offer marking selected Tasks completed as part of the same flow.

### 9.2 Project-Owned Routines

For either terminal Project transition:

```txt
Project ACTIVE → COMPLETED | STOPPED
→ active Project-owned Routines → STOPPED
```

A Routine survives only if the user explicitly changes its ownership before the Project transition.

### 9.3 Atomic Product Meaning

The Project transition, Task resolution choices, and automatic Routine stop are one effective product operation.

Technical transaction behavior belongs to Discussion 019.

### 9.4 Same-Day Occurrence

Current proposal:

When a Project completes or stops on local date D:

- occurrences before D remain historical
- dates after D are no longer eligible
- existing same-day DONE or MISSED occurrence remains unchanged
- existing same-day PENDING occurrence remains a scheduled fact for D
- the user may still complete it that day
- if unresolved after D, it becomes MISSED

The terminal Project transition stops future dates after D rather than rewriting today's existing expectation.

Claude previously considered this semantically coherent. It remains open for final confirmation.

### 9.5 Known Future Projections

After a Project-owned Routine stops:

- future projections must not appear as executable work
- purely technical, untouched future records may be removed
- interacted or historical records must not be silently rewritten

Exact storage behavior belongs to Discussion 019.

### 9.6 Reopen and Immediate Undo

Current proposal:

- completed or stopped Project is not later reopened as the same active entity
- continuing the effort creates a new active phase or Project
- history remains intact

Open question:

> Should an immediate short-lived Undo be allowed as rollback of an accidental terminal action?

Immediate Undo, if accepted, is an action rollback rather than a later lifecycle transition.

---

## 10. Goal Terminal Lifecycle Effects

### 10.1 Achieve Goal

Achieving a Goal is explicit user confirmation of real-world outcome.

Before finalization, active direct Tasks, Routines, and Projects must be shown and resolved.

Current proposal:

- Projects must complete, stop, or detach
- direct Tasks must complete, drop, or detach
- direct Routines must stop or detach
- nothing active remains owned by an achieved Goal

### 10.2 Abandon Goal

Default proposal:

- direct Tasks are dropped unless explicitly detached
- direct Routines stop unless explicitly detached
- child Projects stop unless explicitly detached

No active child remains under an abandoned Goal.

Claude should review whether Goal terminal cascades belong fully in 015 or require a separate lifecycle discussion while preserving this invariant.

---

## 11. Timezone and Local-Date Rules

### 11.1 Effective Timezone

Execution uses a user-selected IANA timezone.

Examples:

```txt
Asia/Tehran
Europe/Berlin
America/Toronto
```

UTC timestamps may support storage, but daily product meaning is based on local date.

### 11.2 Historical Stability

Timezone changes must not silently rewrite historical scheduled local dates.

Enough context must exist to interpret:

- scheduled local date
- timezone used for that schedule
- UTC event timestamp

Exact fields belong to Discussion 019.

### 11.3 Travel and Timezone Changes

Current proposal:

- future Today assembly uses the newly effective timezone
- future recurrence evaluation uses the new timezone
- past occurrences retain original local-date context
- transition must not create duplicate occurrences
- per-Routine fixed timezone is excluded from MVP

### 11.4 DST

Date-based recurrence remains attached to intended local dates.

Exact instant selection for nonexistent or repeated clock times is deferred to Discussions 019–020.

---

## 12. Reconcile Eligibility

Discussion 015 identifies eligibility only. Discussions 016–017 define grouping and actions.

### 12.1 Task Eligibility

Task may become Reconcile-eligible when:

- its planned date passed unresolved
- it has been carried repeatedly
- it was dropped while supporting active parent work
- it was historically corrected or restored
- a parent terminal action conflicts with it

### 12.2 Routine and Occurrence Eligibility

Routine execution may become Reconcile-eligible when:

- occurrences are repeatedly missed across observed periods
- historical corrections materially change the pattern
- recurrence is repeatedly edited
- parent lifecycle stops the Routine

A single missed occurrence does not necessarily require a large Reconcile or planning intervention.

### 12.3 Reconcile Output as Adaptive Input

Accepted boundary:

- Reconcile may produce explicit decisions such as Carry, Drop, correction, or acknowledgement
- those decisions may become evidence for Adaptive Planning
- Adaptive Planning must evaluate evidence quality and repetition
- absence-contaminated periods reduce confidence
- unresolved inactivity must not be interpreted as confirmed low capacity

The exact pattern model belongs outside Discussion 015.

---

## 13. Stabilized Decisions

The following are currently accepted unless Claude finds a contradiction:

- Today is a derived local-date execution view
- Task and Routine execution remain distinct
- authoritative Task placement is `plannedDate`
- Task Carry is explicit and changes placement, not status
- no hidden midnight Carry exists
- overdue is derived, not a canonical Task status
- unresolved past Tasks become Reconcile-eligible
- Routine executes through RoutineOccurrence
- RoutineOccurrence never carries
- occurrence lifecycle is PENDING → DONE | MISSED
- no canonical SKIPPED or PAUSED in MVP
- MISSED is neutral and non-punitive
- stopped Routine cannot reactivate as the same entity
- Resume creates a new continuation Routine
- recurrence edits apply prospectively from an explicit date
- past occurrences are preserved
- Project completion and stop both stop Project-owned Routines
- terminal parent may not retain active dependent children
- existing same-day occurrence remains a scheduled fact
- future occurrence eligibility ends after the stop boundary
- historical correction is not finalized as a fixed seven-day window
- absence is not execution failure
- Reconcile and Adaptive Planning are separate flows
- Adaptive Planning may consume reliable Reconcile outcomes

---

## 14. Open Product Questions and Current Proposals

### Q1. Is `plannedDate = today` the only Task entry mechanism for Today?

Current proposal: yes. All manual, AI, Carry, and Reconcile paths update the same field.

### Q2. Where are overdue Tasks shown?

Current proposal: separate unresolved/Reconcile area, not mixed indefinitely into today's normal checklist.

### Q3. How far ahead or behind are occurrences physically materialized?

Product semantics are decided; exact horizon is deferred to Discussion 019.

### Q4. What limits historical Task and occurrence correction?

Current proposal: allow beyond seven days, require stronger confirmation and audit for older consequential changes, and defer the exact boundary.

### Q5. May completed or dropped Tasks reactivate as the same entity?

Current proposal: yes as explicit audited correction. Claude should check consistency with Discussion 012.

### Q6. What exact options appear for active Tasks during Project completion or stop?

Current proposal: drop, explicit detach/reparent, or cancel transition. Consider allowing explicit complete for selected Tasks.

### Q7. Is the same-day occurrence rule final?

Current proposal: keep existing same-day occurrence; stop only future dates after that local date.

### Q8. Is immediate Undo allowed for Project terminal action?

Current proposal: possibly yes as bounded rollback, while later reopen remains forbidden.

### Q9. Should Goal terminal cascades remain in 015?

Current proposal: retain execution invariant here; exact UX may receive a later specification.

### Q10. Does MVP need `SKIPPED` or `PAUSED`?

Current proposal: no.

---

## 15. Scenario Checks

### Scenario A — Task Date Passed

```txt
Task planned for Tuesday
Tuesday ends without action
```

Expected:

- Task remains ACTIVE
- Tuesday remains its last planned date
- Task becomes overdue and Reconcile-eligible
- Task does not silently move to Wednesday
- Task does not appear as ordinary Wednesday work unless carried or rescheduled

### Scenario B — Repeated Carry

Expected:

- same Task identity remains
- current plannedDate changes each time
- Carry history is preserved
- repeated Carry may become reliable evidence only when user interaction is known

### Scenario C — Month-Long Absence

Expected:

- missing scheduled occurrences can be reconstructed
- historical facts remain correctable
- absence period is low-confidence Adaptive Planning evidence
- system does not conclude that user capacity is extremely low merely from non-use

### Scenario D — Resume Routine

Expected:

- old Routine remains STOPPED
- old occurrence history remains attached to it
- new active continuation Routine is created
- new recurrence may be edited before confirmation

### Scenario E — Complete Project with Active Work

Expected:

- unresolved Tasks are shown
- user explicitly resolves or detaches them, or cancels completion
- Project-owned Routines stop automatically
- no active child remains under completed Project

### Scenario F — Stop Project Today

Expected:

- existing occurrence today remains resolvable
- if unresolved at day end, it becomes MISSED
- no later occurrence remains executable

### Scenario G — Historical Correction After Long Gap

Expected:

- user can state that an old Task or occurrence was actually Done
- correction time and claimed execution date remain distinct
- original history is auditable
- fixed seven-day expiry does not make history permanently false

---

## 16. Claude Review Resolution So Far

Claude's previous review produced:

```txt
F1 — Blocking: Routine reactivation contradicted one-way lifecycle
F2 — Important: fixed seven-day correction window conflicted with long-gap recovery
F3 — Minor: MISSED needed neutral, non-punitive semantics
```

Integrated resolutions:

- F1 accepted: Resume creates a new continuation Routine
- F2 accepted: seven days is not a final correction boundary
- F3 accepted: MISSED is neutral and non-punitive

Additional clarification from Mahdi:

- Reconcile handles unresolved past execution
- Adaptive Planning designs future workload
- Adaptive Planning may consume reliable Reconcile outcomes
- absence and missing interaction must not dominate adaptation
- repeated patterns and explicit feedback are stronger than one isolated period

---

## 17. Mind Map Impact — Record Only, Do Not Apply Yet

After Discussion 021 consolidation, record:

### User Flow

```txt
Approved execution window
→ Today assembled by local date
→ Task complete / carry / drop
→ RoutineOccurrence done / missed
→ unresolved past execution becomes Reconcile-eligible
→ reliable repeated outcomes may inform later Adaptive Planning
```

### Product Model Execution Notes

- Today is derived, not an entity
- Carry changes Task date, not status
- overdue is derived
- Routine executes through occurrences
- occurrence never carries
- MISSED is neutral
- stopped Routine resumes through a new continuation entity
- terminal Project stops Project-owned Routines
- terminal parent cannot retain active dependent children
- historical facts remain correctable and auditable

### AI Guardrails

- absence is not failure
- missing data is not negative evidence
- one weak period is not a pattern
- major adaptation requires repeated reliable evidence or explicit feedback

No Mind Map changes are applied yet.

---

## 18. Affected Formal Documents — Record Only, Do Not Update Yet

After consolidation, accepted decisions should update or create:

- Today execution specification
- Task lifecycle and Carry specification
- Routine continuation and recurrence-edit specification
- RoutineOccurrence execution and correction specification
- Project and Goal terminal cascade specification
- timezone and local-date specification
- Reconcile eligibility and execution-input specification
- Adaptive Planning evidence-quality guardrails
- data and event specification in Discussion 019
- runtime specification in Discussion 020
- validation plan in Discussion 021
- implementation plan in Discussion 022

Potential ADRs:

- Today as a derived local-date view
- occurrence derivation/materialization semantics
- Routine continuation as a new entity rather than lifecycle reversal
- terminal parent child-resolution behavior
- auditable historical correction model

No formal document is updated before consolidation.

---

## 19. Structured Review Request for Claude

Review this exact revision according to `claude-review-standards.md`.

For each remaining issue, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected decision
4. concrete failure scenario
5. why it conflicts or is incomplete
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus on:

- contradictions with accepted Discussions 012 and 014
- whether `plannedDate` alone can determine Task entry into Today
- whether overdue work is separated from Today coherently
- whether Task correction/restoration conflicts with one-way lifecycle language
- whether Routine continuation as a new entity fully resolves F1
- whether historical correction remains honest without a fixed calendar window
- whether DONE/MISSED without SKIPPED remains truthful
- whether MISSED semantics are sufficiently neutral
- whether occurrence reconstruction after inactivity is deterministic
- whether same-day Project-stop occurrence behavior is coherent
- whether terminal parent child-resolution options are complete and non-destructive
- whether Goal lifecycle effects are correctly scoped
- whether Reconcile and Adaptive Planning responsibilities are cleanly separated
- whether absence and low-confidence evidence are prevented from driving aggressive adaptation
- any remaining rule that cannot be validated deterministically

Do not expand into database tables, API payloads, provider choice, exact Reconcile scoring, exact Adaptive Planning thresholds, visual styling, analytics, or implementation sequence.

This discussion remains open until Claude reviews this exact revision and accepted corrections and final resolution are recorded.
