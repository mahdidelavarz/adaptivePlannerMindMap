# Discussion 024 — Multi-Time Daily Routine Scheduling and Occurrence Semantics

## Status

**OPEN — proposed direction, pending Claude review.**

This discussion is **not yet accepted or closed**.

Nothing in this file is authoritative product behavior until review is completed and the resulting direction is explicitly accepted.

```txt
STATUS = OPEN
CLAUDE_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

Do not update `00-Canvas/Planner-Mindmap.canvas`, formal specifications, implementation plans, or runtime contracts from this file while the discussion remains open.

This discussion was opened after identifying a concrete limitation in the accepted Routine model: the MVP currently assumes one Routine produces at most one RoutineOccurrence per local calendar date, but valid real-world recurring behaviors may need multiple scheduled executions within the same day.

Primary related accepted discussions:

- [[01-Closed-Discussions/012-core-product-model]]
- [[01-Closed-Discussions/012a-temporal-checkpoint-amendment]]
- [[01-Closed-Discussions/013-ai-planning-entry-and-conversation-flow]]
- [[01-Closed-Discussions/014-ai-planning-output-contract]]
- [[01-Closed-Discussions/014a-temporal-checkpoint-planning-draft-amendment]]
- [[01-Closed-Discussions/015-task-and-routine-execution-model]]
- [[01-Closed-Discussions/015a-temporal-checkpoint-execution-amendment]]
- [[01-Closed-Discussions/015b-routine-local-date-and-daily-occurrence-amendment]]
- [[01-Closed-Discussions/016-reconcile-trigger-and-severity]]
- [[01-Closed-Discussions/016a-review-checkpoint-trigger-and-presentation-amendment]]
- [[01-Closed-Discussions/017-ai-reconcile-intelligence-and-actions]]
- [[01-Closed-Discussions/019a-canonical-data-model-and-invariants]]
- [[01-Closed-Discussions/019b-transactions-concurrency-and-idempotency]]
- [[01-Closed-Discussions/019c-events-ai-observability-and-retention]]
- [[01-Closed-Discussions/020a-ai-runtime-boundaries-and-orchestration]]
- [[01-Closed-Discussions/020b-api-and-frontend-state-contracts]]
- [[01-Closed-Discussions/020c-structured-output-reliability-and-cost-controls]]
- [[01-Closed-Discussions/023-persistent-planning-facts-and-rolling-execution-context]]

---

## 1. Problem Statement

The accepted MVP Routine model currently includes this simplifying rule:

```txt
one Routine
→ at most one RoutineOccurrence per local calendar date
```

The persistence identity is correspondingly constrained as:

```txt
UNIQUE(routineId, scheduledLocalDate)
```

This works for examples such as:

```txt
walk every day
study English on weekdays
water plants every Saturday
```

but fails for valid recurring behavior that happens more than once per day.

Example:

```txt
Take medicine three times per day.
```

or the user-facing intent:

```txt
Take this medicine every 8 hours.
```

The previous MVP workaround of representing multiple daily executions as multiple independent Routines creates an incorrect domain model:

```txt
Medicine morning
Medicine afternoon
Medicine night
```

This fragments one real-world behavior into multiple Routine identities, histories, lifecycle states, edits, and continuation lineages.

The problem to solve is therefore:

> One Routine must be able to produce multiple distinct RoutineOccurrences on the same local calendar date.

The broader planner does **not** need to become hour-based.

Goal, Project, and Task may remain day-granularity.

Routine is the intentional sub-day exception because recurrence semantics can require multiple executions within a day.

---

## 2. Explicit Reopening of Earlier Decisions

This discussion intentionally reopens and proposes amendments to earlier accepted rules.

### Discussion 015

The accepted one-occurrence-per-day Routine execution assumption must be revisited.

### Discussion 015B

The local-date effectiveness model remains valuable, but the rule:

```txt
one Routine
→ at most one RoutineOccurrence per local calendar date
```

would no longer remain valid if this discussion is accepted.

### Discussion 019A

The persistence constraint:

```txt
UNIQUE(routineId, scheduledLocalDate)
```

would need an explicit replacement.

This discussion must not silently override those accepted decisions.

If accepted, the affected discussions and formal specifications must be reconciled explicitly.

---

## 3. Scope

This discussion proposes rules for:

1. multiple Routine time slots within one local date,
2. RoutineOccurrence sub-day identity,
3. local-time-based recurrence semantics,
4. catch-up/lazy occurrence generation with multiple daily occurrences,
5. automatic PENDING → MISSED boundaries for timed occurrences,
6. prospective Routine recurrence edits,
7. Today presentation of multiple occurrences from one Routine,
8. Reconcile interpretation of multi-occurrence routines,
9. AI Planning representation of sub-day Routine schedules,
10. migration away from the one-occurrence-per-day assumption.

### Explicitly out of scope

This discussion does not propose:

- hour-level scheduling for Goal,
- hour-level scheduling for Project,
- hour-level scheduling for Task,
- a generic calendar/event system,
- arbitrary cron expressions,
- a universal recurrence language,
- true elapsed-time interval recurrence,
- `EVERY_N_HOURS` as a persisted canonical recurrence type in MVP,
- floating anchor-instant semantics,
- duration-based scheduling across DST,
- a new Routine lifecycle,
- a new RoutineOccurrence lifecycle beyond `PENDING | DONE | MISSED`,
- a new Reconcile severity model from zero,
- Mind Map changes while this discussion is open.

---

## 4. Governing MVP Direction

The proposed MVP direction is intentionally narrower than a full interval scheduler.

Routine remains **local-calendar-based**.

A Routine may define:

```txt
recurrenceDefinition
+
one or more local timesOfDay
```

Example:

```txt
Routine: Medicine
recurrence: DAILY
local times:
- 08:00
- 16:00
- 00:00
```

This is one Routine with three daily execution slots.

The planner therefore supports sub-day execution without making all planning entities hour-based.

---

## 5. Why True EVERY_N_HOURS Is Deferred

A literal elapsed-time recurrence such as:

```txt
EVERY_N_HOURS
intervalHours = 8
anchorInstant = ...
```

introduces additional semantics that the current MVP does not otherwise need:

- elapsed duration versus local wall-clock time,
- anchor instant ownership,
- DST transitions,
- whether `scheduledAt` or local date/time is canonical truth,
- cross-midnight interval continuation,
- different timezone semantics from existing local-date-first recurrence.

These are solvable problems, but they materially expand the temporal model.

For MVP, the product instead models explicit local time slots.

Example user request:

```txt
"Take this medicine every 8 hours."
```

must not silently create an invented anchor.

### If a start time is known

```txt
User:
"Every 8 hours, starting at 08:00."

AI may derive explicit local slots:
08:00
16:00
00:00
```

when that derivation is valid for the intended daily repeating pattern.

### If a start time is not known

```txt
User:
"Every 8 hours."

→ clarification required:
"What time should the first daily dose be?"
```

AI must not silently invent 08:00 or another anchor.

### Product honesty rule

The resulting canonical Routine is not represented as a true elapsed-time `EVERY_N_HOURS` rule.

It is represented as explicit local daily time slots.

If future evidence shows that rolling elapsed-time intervals are required, they must be introduced by a later explicit amendment.

---

## 6. Proposed Routine Timing Contract

Conceptually, an ACTIVE Routine may contain:

```txt
Routine
- recurrenceDefinition
- recurrenceTimezone
- effectiveFromLocalDate
- effectiveUntilLocalDate?
- timesOfDay[]
```

### `timesOfDay[]`

For MVP:

```txt
0..N local wall-clock times
```

Examples:

```txt
[08:00]
[08:00, 20:00]
[00:00, 08:00, 16:00]
```

The exact storage shape is deferred until the product contract is accepted, but semantics must remain:

> Each eligible local date may generate one RoutineOccurrence per applicable local time slot.

### Untimed Routine compatibility

The existing product may still support a Routine with no explicit time-of-day.

If retained, an untimed Routine continues to produce at most one occurrence per eligible local date.

Timed and untimed Routine behavior must not become ambiguous.

This compatibility point requires Claude review.

---

## 7. RoutineOccurrence Identity

The current identity:

```txt
UNIQUE(routineId, scheduledLocalDate)
```

is insufficient when a Routine has multiple daily slots.

Proposed MVP identity:

```txt
RoutineOccurrence
- id
- routineId
- scheduledLocalDate
- scheduledLocalTime?
- status: PENDING | DONE | MISSED
- resolvedAt?
- createdAt
- updatedAt
- version
```

For timed Routine occurrences:

```txt
UNIQUE(
  routineId,
  scheduledLocalDate,
  scheduledLocalTime
)
```

If untimed Routines remain valid, their identity must still prevent duplicate untimed occurrences for the same date.

One possible conceptual rule is:

```txt
one Routine + local date + time-slot identity
→ at most one occurrence
```

The exact database constraint for nullable time requires later technical design, but product identity must be unambiguous before implementation.

---

## 8. Local-Time Semantics

MVP Routine time slots are local wall-clock times under the Routine's accepted timezone semantics.

Example:

```txt
Routine local slot = 08:00
```

means:

> execute at 08:00 in the Routine's applicable local timezone.

The product does not interpret this as "exactly 24 elapsed hours after yesterday's occurrence."

This preserves the project's existing local-date-first temporal model.

True elapsed-time interval semantics remain deferred.

DST edge behavior must still be reviewed for impossible or repeated local clock times, but no separate interval-time source of truth is introduced in MVP.

---

## 9. Occurrence Generation

The existing lazy generation/catch-up model must be explicitly revalidated because it was designed under the one-occurrence-per-date assumption.

The proposed generation model becomes conceptually:

```txt
for each eligible local date:
  determine applicable local time slots
  ensure one occurrence exists per slot
```

Example:

```txt
Routine slots:
08:00
16:00
00:00

3 eligible local dates
→ up to 9 RoutineOccurrences
```

Catch-up logic must not assume one row per date.

### Bounded generation

This discussion does not authorize unbounded future pre-generation.

Existing lazy/bounded generation philosophy remains.

The exact generation horizon should reuse accepted execution mechanics unless multi-time support exposes a concrete contradiction.

---

## 10. PENDING → MISSED Boundary

Timed occurrences make the MISSED boundary more precise.

Proposed MVP rule:

> A PENDING timed RoutineOccurrence becomes MISSED when the next scheduled occurrence of the same Routine is reached, or at the end of the current local day, whichever occurs first.

Example:

```txt
Routine slots:
08:00
16:00

08:00 occurrence:
08:00 → PENDING
15:59 → still PENDING
16:00 → MISSED if unresolved

16:00 occurrence:
16:00 → PENDING
end of local day → MISSED if unresolved
```

This avoids arbitrary grace periods such as "30 minutes late".

It also preserves automatic rather than user-required MISSED resolution.

### No background-job requirement

As with existing lazy temporal behavior, the product does not require a job to mutate status at the exact wall-clock second.

The effective status may be reconciled/materialized when the relevant surface is read or processed, provided the canonical history remains coherent.

This exact mechanism belongs to implementation/runtime design, not this product-level rule.

### Historical correction

Accepted correction semantics remain:

```txt
DONE ↔ MISSED
```

for historical correction where permitted.

Scheduled local date/time never moves as part of a correction.

---

## 11. Recurrence Editing

The accepted prospective Routine edit philosophy remains authoritative.

Default rule:

```txt
recurrence/timing edit
→ applies prospectively
→ effective from next local calendar date by default
```

Example:

Today has already exposed:

```txt
08:00 DONE
16:00 PENDING
20:00 PENDING
```

The user edits tomorrow's Routine slots to:

```txt
09:00
21:00
```

Today's occurrences remain unchanged.

The new schedule begins on the next local date.

Past occurrences are never regenerated.

Already exposed current-day occurrences preserve identity and history.

This reuses the accepted `effectiveFromLocalDate` / prospective-edit model rather than inventing a new same-day rewrite rule.

---

## 12. Today Semantics

Today continues to contain current-day RoutineOccurrences.

With a multi-time Routine, Today may contain multiple execution rows belonging to the same Routine.

Preferred conceptual presentation:

```txt
Medicine
  08:00  DONE
  16:00  PENDING
```

rather than presenting them as unrelated Routines.

Each occurrence retains its own execution state and action:

```txt
PENDING → DONE | MISSED
```

No Carry, Drop, or Backlog is introduced for RoutineOccurrence.

### Date boundary

A slot at:

```txt
00:00
```

belongs to the local calendar date on which 00:00 occurs.

It must not remain grouped under the previous day's Today merely because another slot of the same Routine occurred earlier.

---

## 13. Reconcile and Severity

Multi-time Routines create a counting risk.

Example:

```txt
Routine A
3 expected occurrences on one day
3 missed
```

Raw occurrence count alone would treat this as three independent failures and could artificially amplify severity relative to a once-daily Routine.

The proposed direction is to preserve both:

```txt
raw missed occurrence count
+
affected routine-day count
```

Example:

```txt
missedOccurrences = 3
affectedRoutineDays = 1
```

Severity should not treat three missed slots from one Routine on one local date as equivalent to three unrelated Routine/day failures.

Existing ratio-based Routine mismatch logic should continue to use observed occurrence data where appropriate.

Discussion 024 does not redesign Reconcile from zero.

It requires explicit reconciliation of any Discussion 016/017 metric that previously assumed one occurrence per Routine per day.

---

## 14. AI Planning Contract

If AI proposes a timed Routine, time slots must be visible and reviewable.

Examples:

```txt
Routine proposal:
Take medicine
DAILY
08:00
16:00
00:00
```

The AI must not create multiple Routine identities merely to represent multiple daily slots.

### "Every N hours" natural language

For MVP:

```txt
user says every N hours
+
start time known
→ AI may propose explicit local daily slots when representable
```

If required timing information is missing and the answer materially changes the Routine schedule:

```txt
→ clarification
```

This is consistent with Discussion 013's rule that high-impact recurrence ambiguity requires clarification rather than silent invention.

### Discussion 023 interaction

Routine timing information has a canonical home.

Therefore:

```txt
Routine recurrence/time slots
→ canonical Routine fields
→ NOT PlanningFact
```

Do not duplicate Routine timing into persistent planning context.

---

## 15. Manual Creation and Quick Create

Manual Routine creation must be able to represent the same accepted timing semantics as AI-created Routines.

AI must not gain a timing capability unavailable to manual users.

If multi-time slots are accepted, Routine creation/edit surfaces need a way to add, remove, and review multiple local times without turning the form into a generic calendar editor.

Detailed visual design remains out of scope until the product model is accepted.

---

## 16. Lifecycle and Parent Terminal Behavior

Routine lifecycle remains:

```txt
ACTIVE → STOPPED
```

Multi-time scheduling does not introduce PAUSED.

Project terminal cascades still stop Project-owned Routines according to accepted rules.

Historical RoutineOccurrences remain preserved.

Multiple occurrences on one day do not alter parent ownership semantics.

---

## 17. Persistence and Concurrency Impact

If accepted, persistence specifications must be amended to support multi-time occurrence identity.

At minimum, later technical work must address:

- storage of Routine time slots,
- occurrence uniqueness with sub-day identity,
- same-user ownership invariants,
- idempotent lazy generation of multiple slots,
- stale-write behavior for recurrence/timing edits,
- history preservation for exposed occurrences.

No database migration is authorized by this open discussion yet.

---

## 18. Migration / Compatibility

Existing once-daily Routines must remain representable.

A future migration should not require inventing arbitrary times for existing untimed Routines.

Potential compatibility model:

```txt
existing Routine with no time slot
→ remains untimed once-per-day Routine
```

New timed multi-slot behavior is additive at the Routine level while explicitly replacing the global one-occurrence-per-date uniqueness assumption.

Exact migration details are deferred until acceptance.

---

## 19. Proposed Non-Goals and Guardrails

Do not:

- add hour granularity to Task merely because Routine needs it,
- create a new Hour entity,
- create separate Routines for each time slot of one real-world behavior,
- silently invent a start time for "every 8 hours",
- persist `EVERY_N_HOURS` while implementing only fixed local slots,
- claim elapsed-time interval semantics when the product stores wall-clock slots,
- replace exposed same-day occurrences after a Routine edit,
- regenerate past occurrence history,
- multiply Reconcile severity naively by raw same-day occurrence count,
- require a background scheduler merely to make missed status conceptually correct,
- add PAUSED to Routine,
- update the Mind Map while this discussion is open.

---

## 20. Open Questions for Claude Review

Claude should review this proposed direction specifically for contradictions, missing states, ambiguity, edge cases, and conflicts with accepted Discussions 012–023.

Please do **not** redesign the product from zero or reject multi-time Routine support merely to preserve the old MVP simplification.

### Recurrence model

1. Is fixed local `timesOfDay[]` sufficient for the identified MVP problem?
2. Is deferring true elapsed-time `EVERY_N_HOURS` coherent, or does converting user language into explicit daily slots create hidden semantic loss that needs stronger disclosure?
3. Should an untimed Routine remain valid alongside timed Routines?
4. Do recurrence patterns such as `N_TIMES_PER_WEEK` interact ambiguously with multiple `timesOfDay` values?

### Occurrence identity

5. Is `routineId + scheduledLocalDate + scheduledLocalTime` sufficient as product identity for timed occurrences?
6. If untimed Routines remain, what is the cleanest identity rule without creating ambiguous nullable-time uniqueness semantics?
7. Are there DST repeated/nonexistent local-time edge cases that require product-level rules even without true interval scheduling?

### Generation and status

8. Does existing lazy catch-up generation contain any hidden one-occurrence-per-date assumptions beyond the known uniqueness constraint?
9. Is `next occurrence or local-day end, whichever first` a coherent automatic MISSED boundary?
10. Does this create edge cases for adjacent or duplicate time slots?
11. Should duplicate time slots be forbidden by Routine validation?
12. How should a 00:00 slot interact with previous-day grouping and MISSED calculation?

### Editing and history

13. Is next-local-date prospective application sufficient for all timing edits in MVP?
14. Do already generated but never exposed future occurrences need deletion/re-generation under the existing 019B rules?
15. Are current-day PENDING occurrences correctly preserved even if the user edits the Routine early in the day?

### Today / Reconcile

16. Should Today group multiple occurrences under one Routine by default, or is independent list ordering by time more important?
17. Which exact Discussion 016 severity inputs must switch from raw occurrence counts to routine-day-aware metrics?
18. Is keeping both `missedOccurrences` and `affectedRoutineDays` enough to preserve signal without unfair amplification?
19. Does existing R4 routine mismatch ratio already handle multi-occurrence schedules correctly once occurrence generation is fixed?

### AI and authority

20. Is clarification of start time sufficient for natural-language "every N hours" requests in MVP?
21. When explicit time slots derived from user input cross midnight, is the resulting local-day schedule unambiguous to review?
22. Should AI explicitly disclose that the MVP converted interval wording into fixed local clock times rather than preserving elapsed-time semantics?

### Reopened decisions

23. Are Discussions 015, 015B, and 019A the complete set of authoritative sources that must be explicitly amended for the one-occurrence-per-day change?
24. Are there any accepted 016/017/019B/020 contracts that also contain hidden one-occurrence-per-day assumptions and must be named explicitly before closure?

---

## 21. Review Standard

Claude review should classify findings as:

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

Review should prioritize:

- explicit reconciliation of reopened decisions,
- occurrence identity correctness,
- local-date/time consistency,
- DST edge cases,
- catch-up generation correctness,
- PENDING/MISSED semantics,
- history preservation,
- severity amplification,
- AI intent honesty,
- avoiding unnecessary interval-engine complexity.

---

## 22. Current Proposed Direction Summary

Pending review, the proposed direction is:

```txt
Goal / Project / Task
→ remain day-granularity

Routine
→ may define multiple local times per eligible local date

RoutineOccurrence
→ one occurrence per Routine + local date + time slot

true EVERY_N_HOURS elapsed-time recurrence
→ deferred from MVP
```

Execution example:

```txt
Medicine Routine
DAILY
00:00
08:00
16:00

Today:
08:00 DONE
16:00 PENDING
```

Generation:

```txt
eligible local dates
×
applicable local time slots
→ idempotent bounded RoutineOccurrences
```

MISSED:

```txt
PENDING occurrence
→ MISSED when next same-Routine occurrence is reached
   OR local day ends
   whichever occurs first
```

Editing:

```txt
Routine recurrence/time edit
→ prospective
→ next local date by default
→ current-day exposed occurrences preserved
```

Reconcile:

```txt
preserve raw occurrence evidence
+
avoid severity inflation using routine-day-aware aggregation
```

AI:

```txt
"every N hours"
→ do not invent anchor
→ clarify start time when missing
→ propose explicit local slots
→ disclose fixed-local-time semantics where needed
```

---

## 23. Mind Map Impact — NOT YET APPLIED

If Discussion 024 is later accepted, likely Mind Map areas affected include:

### Product Model

Potential changes:

- Routine supports multiple local time slots,
- RoutineOccurrence gains sub-day schedule identity,
- one-occurrence-per-local-date rule removed,
- Goal/Project/Task remain day-granularity.

### MVP Core Loop

Potential changes:

```txt
Routine definition
→ eligible local date
→ one or more local time slots
→ multiple RoutineOccurrences
→ Today execution
→ Reconcile if unresolved/missed
```

### AI Responsibilities

Potential additions:

- propose multiple local time slots,
- clarify missing start time for interval-like user language,
- avoid claiming unsupported elapsed-time semantics.

### AI Guardrails

Potential additions:

- no invented interval anchor,
- no fake multiple Routine identities for one multi-time behavior,
- no silent interval-to-calendar semantic change,
- no PlanningFact duplication for Routine timing.

### Data / Events

Potential changes:

- occurrence identity includes sub-day slot,
- generation/catch-up works per eligible slot,
- multi-occurrence history remains distinct.

**Do not apply any of these changes while Discussion 024 remains OPEN.**

---

## 24. Affected Formal Documents — NOT YET APPLIED

If accepted, Discussion 024 may require explicit amendments to:

- Routine recurrence product contract,
- RoutineOccurrence persistence model,
- execution/catch-up specification,
- Routine edit semantics,
- Today execution specification,
- Reconcile severity/ratio specification,
- PlanningDraft Routine fields,
- AI context/output validation,
- database uniqueness and generation constraints,
- Mind Map and implementation plan.

No formal document is changed by this discussion yet.

---

## 25. Closure Condition

Discussion 024 may be closed only after:

1. Claude review is completed,
2. blocking and important findings are resolved,
3. the final product direction is explicitly accepted,
4. all reopened one-occurrence-per-day assumptions are identified and reconciled,
5. Today/Reconcile/AI consequences are coherent,
6. Mind Map impact is recorded for a separate application step.

Until then:

```txt
STATUS = OPEN
CLAUDE_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

---

# خلاصهٔ فارسی

Discussion 024 محدودیت مهم مدل فعلی Routine را بررسی می‌کند. در مدل پذیرفته‌شدهٔ قبلی، هر Routine در هر تاریخ محلی حداکثر یک RoutineOccurrence داشت. این برای رفتارهایی که چند بار در روز انجام می‌شوند کافی نیست، مثل مصرف یک دارو در ساعت‌های ۰۰:۰۰، ۰۸:۰۰ و ۱۶:۰۰.

جهت پیشنهادی این است که Goal، Project و Task همچنان در سطح روز باقی بمانند، ولی Routine بتواند چند زمان محلی در هر روز واجد شرایط داشته باشد. در نتیجه RoutineOccurrence با ترکیب Routine، تاریخ محلی و time slot از بقیهٔ occurrenceها متمایز می‌شود.

برای MVP، recurrence واقعی از نوع `EVERY_N_HOURS` با anchor شناور و elapsed-time semantics عمداً به تعویق می‌افتد. اگر کاربر بگوید «هر ۸ ساعت»، AI باید در صورت نبودن ساعت شروع سؤال روشن‌کننده بپرسد و سپس زمان‌های محلی صریح پیشنهاد دهد. نباید ساعت شروع را حدس بزند یا وانمود کند سیستم interval واقعی ذخیره کرده است.

مدل catch-up باید از «یک occurrence در هر روز» به «هر تاریخ واجد شرایط × هر time slot» گسترش پیدا کند. پیشنهاد فعلی برای MISSED این است که occurrence حل‌نشده هنگام رسیدن occurrence بعدی همان Routine یا پایان روز محلی، هرکدام زودتر رخ دهد، MISSED محسوب شود.

ویرایش زمان‌بندی Routine همچنان prospective است و به‌طور پیش‌فرض از روز محلی بعد اثر می‌کند. occurrenceهای امروز که قبلاً ایجاد/نمایش داده شده‌اند دست‌نخورده می‌مانند.

در Reconcile باید از افزایش مصنوعی severity به‌خاطر چند occurrence در یک روز جلوگیری شود. raw occurrence evidence حفظ می‌شود، ولی aggregation در سطح routine-day نیز لازم است.

این Discussion عمداً تصمیم‌های قبلی 015، 015B و 019A دربارهٔ one-occurrence-per-day را reopen می‌کند، ولی هنوز باز است، برای review کلاد نوشته شده، در Mind Map اعمال نشده و هیچ implementation نهایی از آن مجاز نیست.
