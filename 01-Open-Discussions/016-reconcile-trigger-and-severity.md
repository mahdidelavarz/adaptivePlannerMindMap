# Discussion 016 — Reconcile Trigger and Severity

## Status

Open for GPT × Claude review.

This revision contains GPT's proposed product rules for Reconcile eligibility, presentation severity, skip behavior, and same-day retrigger control. It depends on the execution semantics accepted in Discussion 015 and must remain open until Claude reviews this exact version and Mahdi resolves any remaining product choices.

Related accepted discussion:

- [[01-Open-Discussions/015-task-and-routine-execution-model]]

---

## 1. Scope

This discussion defines when Reconcile becomes eligible to appear, how prominently it should appear, and how often the product may present it.

It determines:

- the difference between Reconcile eligibility and presentation severity
- which unresolved execution facts can trigger Reconcile
- when Reconcile must not be shown
- how `LIGHT`, `MEDIUM`, and `RECOVERY` differ
- how absence affects context without being treated as failure
- whether Reconcile blocks Today
- skip and remind-later behavior
- same-day presentation and retrigger limits
- when severity is calculated
- the minimum context needed for deterministic validation

Core question:

```txt
When should the product invite the user to resolve past execution,
and how much attention should that invitation demand?
```

---

## 2. Explicitly Out of Scope

This discussion does not define:

- semantic grouping of Reconcile items — Discussion 017
- AI-generated recommendations or explanations — Discussion 017
- exact Carry, Drop, Complete, correction, or bulk-action UX — Discussion 017
- Adaptive Planning workload recommendations or pattern thresholds
- database tables, indexes, transactions, or event persistence — Discussion 019
- API and runtime implementation — Discussion 020
- final validation test plan — Discussion 021
- implementation sequencing — Discussion 022
- exact visual styling, colors, animation, or responsive layout

Discussion 016 answers **when and how strongly Reconcile appears**. Discussion 017 answers **what Reconcile shows and recommends after it is opened**.

---

## 3. Accepted Inputs from Discussion 015

The following execution facts are already accepted and are not reopened here:

- a past-due unresolved Task remains `ACTIVE`
- overdue is a derived condition, not a Task status
- unresolved past Tasks become Reconcile-eligible
- Carry is explicit and repeated Carry may become evidence
- RoutineOccurrence never carries
- past RoutineOccurrences may become `MISSED`
- `MISSED` is a neutral historical fact, not debt or moral failure
- historical corrections may become Reconcile evidence
- parent lifecycle conflicts may require explicit resolution
- Reconcile resolves past execution
- Adaptive Planning proposes future plan changes
- Adaptive Planning may consume reliable Reconcile outcomes
- absence is not failure
- missing interaction is not negative evidence
- one weak period is not a stable behavioral pattern

Reconcile trigger rules must preserve these distinctions.

---

## 4. Core Model: Eligibility Is Not Severity

The product must separate two decisions:

```txt
1. Is there actionable unresolved execution to review?
2. If yes, how prominently should Reconcile be presented?
```

### 4.1 Eligibility

`eligible = true` means at least one execution fact may benefit from an explicit user decision or review.

Eligibility alone does not require a dedicated flow, interruption, modal, or blocking screen.

### 4.2 Presentation Severity

Severity determines presentation weight:

```txt
LIGHT
MEDIUM
RECOVERY
```

Severity is not a diagnosis of the user's discipline, motivation, capacity, or personal worth. It describes only how much unresolved execution context the product needs to surface.

### 4.3 No Universal Numeric Failure Score

MVP severity should use deterministic rule bands rather than an opaque weighted score.

Reasons:

- rules are easier to validate
- edge cases remain understandable
- absence cannot accidentally dominate a hidden score
- the UI can explain why Reconcile appeared
- thresholds can later be calibrated without redefining the product contract

Later implementation may derive an internal score, but user-visible severity must remain explainable through accepted rules.

---

## 5. Reconcile Eligibility

Reconcile becomes eligible when at least one **actionable unresolved execution fact** exists.

### 5.1 Eligible Task Facts

An active Task is eligible when one or more are true:

- its authoritative `plannedDate` is before the current local date
- it has been carried repeatedly and may need a deliberate decision
- it was dropped or historically corrected in a context that may require review
- it conflicts with an attempted parent lifecycle transition
- its ownership became unresolved through an explicit lifecycle operation

A single recently overdue Task is sufficient for eligibility but normally only produces `LIGHT` presentation.

### 5.2 Eligible Routine and Occurrence Facts

Routine execution is eligible when one or more are true:

- recent occurrences form a repeated missed pattern across observed periods
- historical corrections materially change previously understood execution
- recurrence has been repeatedly edited and may no longer fit
- a parent lifecycle transition stops the Routine and user context may need review
- missing execution facts remain genuinely unresolved rather than deterministically classifiable

A single ordinary `MISSED` occurrence is not automatically actionable and does not by itself require Reconcile.

### 5.3 Eligible Structural Conflicts

Reconcile may be eligible when:

- a terminal Project or Goal action encounters active dependent work
- ownership or lifecycle state cannot remain coherent without an explicit decision
- a previously skipped resolution flow leaves a known structural conflict pending

Such conflicts may block the specific lifecycle action, but they do not block access to Today.

### 5.4 Non-Actionable Facts

The following do not independently make Reconcile eligible:

- user absence without actionable unresolved work
- isolated `MISSED` RoutineOccurrences that require no decision
- facts already resolved in the current local day
- deterministic state cleanup completed without user choice
- analytics-only signals
- Adaptive Planning calibration signals without unresolved past execution
- future Tasks or future RoutineOccurrences
- completed or dropped historical work that requires no correction

---

## 6. Deterministic Cleanup Before Severity

Severity must be calculated only after deterministic execution cleanup.

Required order:

```txt
1. determine current local date and effective timezone
2. derive or materialize required RoutineOccurrences
3. convert past unresolved PENDING occurrences to MISSED where applicable
4. remove future projections and non-actionable historical facts
5. identify actionable Reconcile-eligible facts
6. deduplicate facts already handled in the current evaluation boundary
7. calculate severity
```

This prevents raw occurrence counts from inflating severity.

Example:

```txt
30 reconstructed MISSED occurrences
+ no user decision required
≠ 30 Reconcile backlog items
```

A summary of the pattern may still be useful, but each MISSED occurrence must not automatically become a separate recovery action.

---

## 7. Severity Inputs

Severity may consider the following deterministic inputs.

### 7.1 Primary Inputs

- `unresolvedTaskCount`
- `oldestUnresolvedAgeDays`
- `repeatedCarryTaskCount`
- `maximumCarryCountForOneTask`
- `deadlineRiskCount`
- `blockingConflictCount`
- `affectedProjectCount`
- `affectedGoalCount`
- `actionableCorrectionCount`
- `newItemsSinceLastReconcile`

### 7.2 Routine Summary Inputs

- `recentMissedPatternCount`
- `observedOccurrenceCount`
- `occurrenceCorrectionCount`
- `recurrenceEditCount`

Routine inputs must represent patterns or actionable context, not merely the total number of missed dates.

### 7.3 Absence Context

- `absenceDays`
- `activityObservedDuringPeriod`
- `absenceConfidence`

Accepted rule:

```txt
absenceDays modifies context and confidence
absenceDays does not independently create failure severity
```

Long absence may justify a recovery-oriented entry experience only when actionable unresolved work also exists.

---

## 8. Proposed Severity Bands

These are MVP rule bands proposed for Claude review. Exact numeric values may be adjusted while preserving the semantic boundaries.

### 8.1 LIGHT

Use `LIGHT` when all of the following are true:

- actionable unresolved work is small
- backlog is recent
- no blocking structural conflict exists
- no meaningful deadline risk exists
- the situation can be understood without a dedicated recovery session

Proposed examples:

```txt
1–2 unresolved Tasks
oldest age <= 2 local days
no Task carried more than once during the unresolved period
no blocking conflict
no urgent deadline risk
```

Presentation:

- small inline card, banner, or entry point
- fully optional
- does not interrupt Today
- may offer one quick resolution action
- may be dismissed for the current local day

Example message meaning:

> One task from yesterday still needs a decision.

### 8.2 MEDIUM

Use `MEDIUM` when unresolved work requires deliberate review but does not represent broad recovery.

Any of these may qualify:

```txt
3–7 actionable unresolved Tasks
oldest unresolved age between 3 and 7 local days
repeated Carry on one or more Tasks
more than one affected Project or Goal
one or more meaningful deadline risks
multiple related corrections or drops
```

Presentation:

- prominent Reconcile card or page entry
- recommended before normal planning changes
- still skippable
- Today remains accessible
- skipped state may remain visible through a badge or reminder

### 8.3 RECOVERY

Use `RECOVERY` when the unresolved execution context is broad, stale, structurally conflicted, or difficult to resolve safely in one lightweight action.

Any strong condition may qualify:

```txt
8+ actionable unresolved Tasks
oldest unresolved age > 7 local days with active backlog
3+ affected parent contexts
blocking structural conflict
large backlog after meaningful absence
several periods of unresolved Carry or Drop decisions
```

Presentation:

- dedicated recovery entry experience
- chunked rather than overwhelming
- explicit acknowledgement that the user may continue Today first
- non-punitive language
- no claim that absence or backlog proves low ability

`RECOVERY` describes the product's need to help organize unresolved execution. It does not describe the user as failing.

### 8.4 Severity Escalation Priority

A higher band overrides a lower one.

```txt
blocking structural conflict → RECOVERY presentation context
meaningful deadline risk → at least MEDIUM
small recent backlog without risk → LIGHT
```

Claude should review whether structural conflict should always map to `RECOVERY` or instead use a separate blocking flag alongside severity.

---

## 9. Absence Rules

### 9.1 Absence Alone Does Not Trigger Reconcile

```txt
absenceDays > 0
+ no actionable unresolved execution
→ no Reconcile
```

The product may show a neutral welcome-back message, but that is not Reconcile.

### 9.2 Absence with Backlog

When absence and actionable backlog coexist:

- absence explains uncertainty and stale context
- backlog determines eligibility
- severity reflects actionable work, age, risk, and structural spread
- the product should ask for context rather than infer low capacity

### 9.3 Low-Confidence Period

A period with little or no user interaction must be marked conceptually as low-confidence evidence for Adaptive Planning.

Reconcile may still resolve items from that period, but downstream adaptation must distinguish:

```txt
explicit Carry / Drop / Complete decisions
from
unobserved absence and automatically reconstructed MISSED facts
```

---

## 10. Reconcile Must Not Block Today

Accepted proposal:

```txt
Reconcile is non-blocking by default.
```

The user may enter Today even when severity is `RECOVERY`.

Reasons:

- current execution should not be held hostage by past backlog
- forcing recovery first may increase avoidance
- the user may have urgent current work
- Reconcile should reduce friction, not become punishment

Exception:

A structural conflict may block the specific lifecycle action that created it.

Example:

```txt
Project completion with active child Tasks
→ Project completion remains blocked until child resolution
→ Today remains available
```

---

## 11. Skip and Remind-Later Behavior

### 11.1 LIGHT

- may be dismissed for the current local day
- no confirmation dialog required
- unresolved count remains discoverable

### 11.2 MEDIUM

- may be skipped through an explicit action
- product may offer `Review later today` or `Remind me tomorrow`
- Today opens immediately after skip

### 11.3 RECOVERY

- remains skippable
- skip action should explicitly acknowledge that unresolved work remains
- user may choose a later review time or next-open reminder
- no shame, warning language, or artificial urgency unless a real deadline exists

### 11.4 Skip Does Not Resolve Items

Skipping presentation changes only presentation state.

```txt
SKIPPED presentation
≠ reconciled execution facts
```

---

## 12. Trigger Timing

Reconcile eligibility may be evaluated:

- when the app opens
- when Today is opened
- after deterministic occurrence cleanup
- after an execution action creates new unresolved context
- before a parent lifecycle transition that requires child resolution
- when the user manually opens Reconcile

The product should not run a full visible Reconcile presentation after every execution event.

Recommended default:

```txt
passive evaluation on app or Today open
→ present at most one primary prompt per local day
```

---

## 13. Same-Day Presentation and Retrigger

### 13.1 Primary Prompt Limit

The product may automatically present at most one primary Reconcile prompt per user-local day.

This applies whether the user:

- completed Reconcile
- skipped it
- dismissed a `LIGHT` prompt
- opened Today directly

### 13.2 Badge and Count Updates

New unresolved items may update:

- badge count
- Reconcile page content
- non-interruptive status text

They must not automatically reopen the primary prompt in the same local day.

### 13.3 Same-Day Retrigger Exceptions

A second interruptive presentation is allowed only when:

- a new blocking structural conflict appears
- the user explicitly asks to open Reconcile
- a real deadline or safety-critical execution condition emerges

Normal new overdue work is not enough for same-day forced retrigger.

### 13.4 Next-Day Re-evaluation

On the next local day:

- eligibility is recalculated from current facts
- prior skip does not permanently suppress Reconcile
- severity may increase, decrease, or disappear
- already resolved items must not be shown again

---

## 14. Reconcile Session Boundary

The product must distinguish the evaluated snapshot from later changes.

Conceptual boundary:

```txt
session evaluated facts
+ session decisions
+ facts created after evaluation
```

New facts after completion become `newItemsSinceLastReconcile` and are eligible for a later session or manual review.

The system must not claim the backlog is fully resolved if new facts appeared after the evaluated boundary.

Exact persistence and transaction semantics belong to Discussion 019.

---

## 15. Context Required for Validation

The following conceptual context must be available when severity is calculated or a presentation decision is validated.

```txt
ReconcileContext
- evaluatedLocalDate
- effectiveTimezone
- evaluatedAt
- triggerReasons[]
- severity
- actionableUnresolvedTaskCount
- oldestUnresolvedAgeDays
- repeatedCarryTaskCount
- maximumCarryCountForOneTask
- deadlineRiskCount
- blockingConflictCount
- affectedProjectCount
- affectedGoalCount
- actionableRoutinePatternCount
- actionableCorrectionCount
- absenceDays?
- activityObservedDuringAbsence
- evidenceConfidence
- lastPresentedLocalDate?
- lastCompletedAt?
- lastSkippedAt?
- evaluatedBoundary
- newItemsSinceLastReconcile
- presentationState
```

Conceptual `presentationState`:

```txt
NOT_PRESENTED
PRESENTED
DISMISSED
SKIPPED
STARTED
COMPLETED
```

These are product concepts for deterministic behavior. Exact fields and storage belong to Discussion 019.

---

## 16. Direct Answers to Original Questions

### 1. Is one unresolved Task from yesterday enough?

Yes for eligibility. Normally it produces `LIGHT`, non-blocking presentation.

### 2. Does absence create a separate entry path?

Absence is separate context but not an independent Reconcile trigger. Absence plus actionable backlog may influence `RECOVERY` presentation.

### 3. When must Reconcile not be shown?

When no actionable unresolved execution exists, only deterministic or analytics-only facts exist, or all eligible facts were already handled for the relevant boundary.

### 4. Can the user skip and enter Today?

Yes. Reconcile is non-blocking by default at every severity.

### 5. How often may it trigger in one local day?

At most one automatic primary presentation, except a new blocking conflict or explicit user request.

### 6. Which inputs determine severity?

Actionable count, age, Carry repetition, deadline risk, structural conflicts, affected parents, actionable corrections, routine patterns, and absence context.

### 7. What are the exact bands?

Proposed MVP rule bands are defined in Section 8 and remain open for Claude review.

### 8. Severity before or after occurrence cleanup?

After deterministic cleanup and removal of non-actionable facts.

### 9. Does Reconcile block Today?

No. Only the specific conflicting lifecycle action may be blocked.

### 10. What if new overdue work appears after completion?

Update counts and include it in the next session. Do not automatically reopen the primary prompt the same day.

### 11. How is same-day retrigger prevented?

Through local-date presentation state and an evaluated session boundary.

### 12. Which context fields are required?

The conceptual fields in Section 15.

---

## 17. Scenario Checks

### Scenario A — One Task from Yesterday

```txt
1 active Task
plannedDate = yesterday
no deadline risk
no prior Carry
```

Expected:

- Reconcile eligible
- severity `LIGHT`
- small optional entry point
- Today remains accessible
- dismissal suppresses another primary prompt that day

### Scenario B — One-Month Absence, No Backlog

Expected:

- no Reconcile
- optional neutral welcome-back UX may appear elsewhere
- no failure interpretation
- no automatic workload reduction signal

### Scenario C — One-Month Absence with Ten Old Tasks

Expected:

- Reconcile eligible
- likely `RECOVERY`
- severity comes from actionable backlog and age, not absence alone
- recovery experience is chunked and skippable
- evidence confidence for Adaptive Planning remains low until user decisions clarify the period

### Scenario D — Many Reconstructed MISSED Occurrences

```txt
20 reconstructed MISSED occurrences
0 unresolved Tasks
no actionable correction request
```

Expected:

- deterministic cleanup completes
- no twenty-item Reconcile backlog
- optionally show a summarized pattern only when it becomes actionable
- absence is not interpreted as low capacity

### Scenario E — Repeated Carry

```txt
Task carried three times
still ACTIVE
```

Expected:

- Reconcile eligible
- at least `MEDIUM` if the repeated pattern requires deliberate review
- user may still skip
- explicit later decisions may become reliable Adaptive Planning evidence

### Scenario F — New Task Becomes Overdue After Reconcile

Expected:

- badge/count updates
- no second automatic primary prompt that local day
- item appears in manual Reconcile or next-day evaluation

### Scenario G — Project Completion Conflict

Expected:

- specific Project completion remains blocked
- child resolution entry may open immediately
- Today itself remains accessible
- blocking conflict contributes strong severity context

---

## 18. Open Product Questions for Claude

### Q1. Are the proposed numeric bands coherent?

Specifically review:

- `LIGHT`: 1–2 Tasks and <= 2 days
- `MEDIUM`: 3–7 Tasks or 3–7 days
- `RECOVERY`: 8+ Tasks or > 7 days with active backlog

The numbers are proposed MVP thresholds, not universal human-capacity judgments.

### Q2. Should blocking structural conflict be a separate flag?

Current proposal allows it to force `RECOVERY` context. Alternative:

```txt
severity = LIGHT | MEDIUM | RECOVERY
blockingConflict = true | false
```

This may be conceptually cleaner because blocking applies to a specific lifecycle action, not Today.

### Q3. Should repeated Carry immediately force MEDIUM?

Current proposal: repeated Carry on one Task is enough for deliberate review, but Claude should assess whether two Carries or three Carries should be the deterministic threshold.

### Q4. Should RECOVERY ever be non-skippable?

Current proposal: no. Only the specific conflicting action may be blocked.

### Q5. Is one primary prompt per local day sufficient?

Review whether deadline-risk changes should be an additional retrigger exception beyond blocking conflicts and explicit user action.

### Q6. Are raw MISSED occurrence counts correctly excluded?

Current proposal: only actionable routine patterns contribute meaningfully to severity.

---

## 19. Mind Map Impact — Record Only, Do Not Apply Yet

After consolidation, record the following.

### Product Vision

Add:

- Reconcile reduces the cost of returning after deviation
- past backlog must not block present-day action
- recovery language is neutral and non-punitive
- absence is uncertainty, not proof of failure

### MVP Core Loop

```txt
Open app / Today
→ deterministic execution cleanup
→ collect actionable unresolved facts
→ calculate Reconcile severity
→ optional Reconcile entry
→ Today remains accessible
→ accepted decisions update execution history
```

### User Flow

Add three entry patterns:

```txt
LIGHT
→ inline prompt
→ quick resolve or dismiss

MEDIUM
→ prominent Reconcile entry
→ review now or later

RECOVERY
→ dedicated chunked recovery
→ continue Today or resolve backlog
```

Add same-day suppression:

```txt
primary prompt shown once
→ later facts update badge only
→ next local day recalculates
```

### AI Responsibilities

- AI must not decide trigger eligibility before deterministic cleanup
- AI may explain severity from explicit rule inputs
- AI must not use absence alone as evidence of poor performance
- AI recommendations remain deferred to Discussion 017
- AI must distinguish observed decisions from unobserved inactivity

### AI Guardrails

Add:

- no Reconcile from absence alone
- no Reconcile from raw MISSED count alone
- no hidden interpretation of backlog as low capability
- no blocking Today because of Reconcile severity
- no repeated automatic primary prompt in one local day
- no automatic resolution when user skips
- no opaque severity without explainable trigger reasons

### Data Events

Record candidates for Discussion 019:

```txt
RECONCILE_ELIGIBILITY_EVALUATED
RECONCILE_PRESENTED
RECONCILE_DISMISSED
RECONCILE_SKIPPED
RECONCILE_STARTED
RECONCILE_COMPLETED
RECONCILE_SEVERITY_CHANGED
RECONCILE_NEW_ITEMS_DETECTED
RECONCILE_BLOCKING_CONFLICT_DETECTED
```

Each evaluation event should later preserve the evaluated boundary and summarized reason inputs.

### Traction Metrics

Valid product metrics:

- percentage of eligible sessions where user starts Reconcile
- completion rate by severity band
- median actionable items resolved per session
- percentage of users who continue Today after skipping
- backlog age reduction after Reconcile
- same-day repeated-prompt rate, expected near zero
- return-to-execution rate after `RECOVERY`

Guardrail metrics:

- abandonment after Reconcile presentation
- excessive prompt frequency
- percentage of Recovery sessions caused mainly by absence rather than actionable backlog
- percentage of reconstructed MISSED facts incorrectly surfaced as individual actions

Do not use raw completion rate as proof that Reconcile is beneficial without retention and return-to-execution context.

### Current Decisions

Record:

- eligibility and severity are separate
- Reconcile requires actionable unresolved execution
- one recent unresolved Task produces Light eligibility
- severity is rule-based for MVP
- severity is calculated after deterministic cleanup
- absence alone does not trigger Reconcile
- Today remains accessible at every severity
- automatic primary presentation is limited to once per local day
- same-day new items update counts without forced reopening
- skip changes presentation state only

### Open Questions

Keep for Claude and later resolution:

- final numeric thresholds for each band
- whether blocking conflict is separate from severity
- exact repeated-Carry threshold
- exact deadline-risk retrigger rule
- whether any future exceptional case may block Today

Remove questions about whether absence alone means Recovery or whether every MISSED occurrence becomes a Reconcile item.

---

## 20. Affected Formal Documents — Record Only, Do Not Update Yet

After consolidation, accepted decisions should update or create:

- Reconcile eligibility specification
- Reconcile severity and presentation policy
- Today/Reconcile coexistence specification
- same-day prompt suppression specification
- Reconcile context and evaluation-boundary specification
- Adaptive Planning evidence-quality guardrails
- Reconcile grouping and recommendation dependency for Discussion 017
- data and event specification in Discussion 019
- runtime evaluation specification in Discussion 020
- validation plan in Discussion 021
- implementation plan in Discussion 022

Potential ADRs:

- rule-based explainable Reconcile severity for MVP
- non-blocking Reconcile and Today access
- absence as confidence context rather than failure signal
- once-per-local-day primary presentation policy

No Mind Map or formal document is updated before consolidation.

---

## 21. Structured Review Request for Claude

Review this exact revision for product coherence and closure readiness.

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

- contradictions with accepted Discussion 015
- eligibility versus severity separation
- whether one recent unresolved Task should produce Light
- whether absence is prevented from dominating severity
- whether deterministic cleanup happens at the correct boundary
- whether raw MISSED counts are excluded correctly
- whether severity bands are deterministic and explainable
- whether numeric thresholds create bad edge cases
- whether Recovery remaining skippable is coherent
- whether blocking conflicts should be a separate flag
- whether once-per-local-day presentation is sufficient
- whether same-day new items are handled honestly
- whether required context is enough for deterministic validation
- whether Reconcile and Adaptive Planning remain cleanly separated

Do not expand into semantic grouping, AI recommendations, database schemas, API payloads, visual styling, exact Adaptive Planning thresholds, analytics implementation, or implementation sequencing.

This discussion remains open until Claude reviews this exact revision, Mahdi resolves accepted findings, and final Review Resolution, Mind Map Impact, and Affected Formal Documents are recorded.