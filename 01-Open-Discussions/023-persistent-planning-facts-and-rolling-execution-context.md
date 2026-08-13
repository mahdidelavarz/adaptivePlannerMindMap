# Discussion 023 — Persistent Planning Facts and Rolling Execution Context

## Status

**OPEN — Claude review round 1 completed; findings integrated; final re-review pending.**

This discussion is **not yet accepted or closed**.

Nothing in this file is authoritative product behavior until the final review is completed and the resulting direction is explicitly accepted.

```txt
STATUS = OPEN
CLAUDE_REVIEW_ROUND_1 = COMPLETED
ROUND_1_FINDINGS = INTEGRATED
FINAL_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

Do not update `00-Canvas/Planner-Mindmap.canvas`, formal specifications, implementation plans, or runtime contracts from this file while the discussion remains open.

This discussion was opened after identifying a gap in the accepted seven-day AI Planning model: the product defines long-term Goals and rolling weekly execution, but it does not yet define which future-relevant information from the original planning conversation survives approval and becomes available to later weekly planning windows.

Primary related accepted discussions:

- [[01-Closed-Discussions/012-core-product-model]]
- [[01-Closed-Discussions/012a-temporal-checkpoint-amendment]]
- [[01-Closed-Discussions/013-ai-planning-entry-and-conversation-flow]]
- [[01-Closed-Discussions/014-ai-planning-output-contract]]
- [[01-Closed-Discussions/014a-temporal-checkpoint-planning-draft-amendment]]
- [[01-Closed-Discussions/016-reconcile-trigger-and-severity]]
- [[01-Closed-Discussions/016a-review-checkpoint-trigger-and-presentation-amendment]]
- [[01-Closed-Discussions/017-ai-reconcile-intelligence-and-actions]]
- [[01-Closed-Discussions/018-action-permissions-trust-and-reversibility]]
- [[01-Closed-Discussions/018a-ai-failure-privacy-domain-and-hostile-input-resolution]]
- [[01-Closed-Discussions/019a-canonical-data-model-and-invariants]]
- [[01-Closed-Discussions/019c-events-ai-observability-and-retention]]
- [[01-Closed-Discussions/020a-ai-runtime-boundaries-and-orchestration]]
- [[01-Closed-Discussions/020b-api-and-frontend-state-contracts]]
- [[01-Closed-Discussions/020c-structured-output-reliability-and-cost-controls]]
- [[01-Closed-Discussions/022-updated-mvp-implementation-plan]]

---

## 1. Problem Statement

The accepted AI Planning model intentionally separates:

```txt
long-term Goal / Project direction
+
maximum seven-day detailed execution
```

A user may request a one-year plan while the AI creates only the first seven days of detailed Task and Routine execution.

Discussion 014 already defines:

```txt
Week 1
→ calibration period
→ explicit first continuation decision
→ later signal-based continuation
```

However, a missing contract remains:

> When Week 2, Week 3, or Week 20 is generated, what durable product context preserves the important future-relevant facts the user gave during the original planning conversation?

Example:

```txt
User goal:
Reach English B2 within one year.

Clarification answers:
- current level is A2
- about 45 minutes are available per study day
- Friday must remain completely free from study
- only a phone is available
- conversation is more important than grammar-heavy study
- IELTS preparation is not part of the goal
```

The first PlanningDraft may use all of this correctly.

After approval, the raw conversation or temporary PlanningDraft must not become permanent product memory by accident. But if none of the future-relevant information survives, later weekly planning can lose important constraints, resources, preferences, starting state, or explicit exclusions.

The product must not depend on model-session memory for correctness.

Core requirement:

> Weekly Planning AI must receive durable planning context from product state, not from an assumption that the model remembers an earlier conversation.

---

## 2. Scope

This discussion defines a proposed contract for:

1. which information from Planning conversation may survive approval,
2. which information must instead be written into existing canonical entity fields,
3. which information must remain temporary and disappear with the draft,
4. structured `PlanningFact` records for future-relevant user facts that have no existing canonical home,
5. HARD, SOFT, and INFORMATIONAL semantics,
6. a closed vocabulary for deterministically enforceable HARD constraints,
7. AI extraction and explicit user review of proposed Planning Facts,
8. editing, removing, expiration, and reconfirmation,
9. deterministic validation of supported HARD constraints,
10. assembly of context for rolling Week-N planning,
11. regeneration of long-term roadmap narrative instead of persistence of stale AI strategy text,
12. privacy, retention, provenance, and observability inheritance,
13. Goal and standalone Project scope.

### Explicitly out of scope

This discussion does not propose:

- a canonical `Plan` entity,
- persistence of the full AI conversation as planning truth,
- persistence of arbitrary AI strategy narrative,
- a global user planning-profile system,
- a universal constraint language,
- deterministic enforcement of every natural-language preference,
- new Task time-of-day or duration fields,
- a new Reconcile severity model,
- a new Goal or Project lifecycle,
- hard blocking of manual user edits from Planning Facts,
- Mind Map changes while this discussion is open.

---

## 3. Existing Accepted Decisions That Must Remain Intact

### 3.1 No canonical Plan entity

Canonical MVP work entities remain:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

Planning Facts are subordinate durable planning context, not another top-level work entity and not a persisted Plan.

### 3.2 Seven-day detailed execution horizon

Long-lived Goals, Projects, and Routines may span months or years.

Detailed AI-generated Task execution remains limited to no more than seven consecutive local calendar days per execution window.

Discussion 023 does not authorize a 365-day scheduled Task backlog.

### 3.3 Approval boundary

AI proposes.

The user approves.

No durable Planning Fact may silently become trusted state because the model extracted or inferred it.

### 3.4 Derive instead of duplicate

Existing source-of-truth discipline remains mandatory.

```txt
FirstWeekEntry.date → derived from Task.plannedDate
nextTemporalCheckpoint → derived
carryCount → derived from events
PlanningFact.category → derived from factType
```

Planning Facts must not duplicate information that already has the correct canonical home.

---

## 4. Governing Principle

Do not persist the planning conversation.

Do not persist AI strategy narrative as long-term truth.

Persist only reviewed, future-relevant, structured user planning facts that:

```txt
1. materially affect future planning,
2. do not already have an appropriate canonical field,
3. can be represented without inventing psychological or behavioral conclusions,
4. have explicit provenance,
5. are visible, editable, and removable by the user.
```

Every later weekly planning context is rebuilt from:

```txt
canonical current state
+
confirmed applicable Planning Facts
+
recent bounded execution evidence
+
relevant scoped deterministic Reconcile facts
+
current local temporal context
```

Model memory is never a correctness dependency.

---

## 5. Three-Way Information Placement Rule

Every potentially persistent planning detail must pass this classification.

### Case A — Existing canonical home exists

Store it only in the canonical field.

Examples:

```txt
Goal target date → Goal.targetDate
Task execution date → Task.plannedDate
Routine weekday pattern → Routine.recurrenceDefinition
Project completion meaning → Project.completionMeaning
Project target date → Project.targetDate
```

Do not duplicate the same information in Planning Facts.

### Case B — Future-relevant and no canonical home exists

AI may propose a structured Planning Fact for explicit user review.

Examples:

```txt
current English level is A2
Friday must remain unavailable for this planning scope
only a phone is available
conversation-focused learning is preferred
IELTS preparation is explicitly excluded
```

### Case C — Draft-local only

Keep it only inside the Planning flow and discard it with the draft lifecycle.

Core rule:

> Persistence is justified by future planning value, not merely because something appeared in conversation.

### Deterministic no-duplication gate

Before proposed Planning Facts become reviewable, the Planning validation pipeline must check whether each proposed fact belongs in an existing canonical field proposed in the same draft.

```txt
proposed PlanningFact overlaps canonical proposal field
→ do not persist both
→ canonical field remains source of truth
```

Where the mapping is deterministically known, the PlanningFact proposal must be rejected or folded into the canonical proposal before review.

The AI may not create a parallel fact merely to preserve conversational wording.

---

## 6. Planning Fact Scope

Round-one review identified that Goal-only scope would recreate the same continuity gap for standalone Projects, which are explicitly supported by Discussion 013.

For MVP, a durable Planning Fact is scoped to exactly one of:

```txt
Goal
OR
standalone Project
```

Conceptually:

```txt
PlanningFact
- goalId?
- projectId?

constraint:
exactly one is non-null
```

Rules:

- Goal-owned Project planning normally consumes the parent Goal's Planning Facts rather than duplicating them at the Project level.
- standalone Project may own its own Planning Facts.
- Task and Routine do not own persistent Planning Facts in MVP.
- a one-off standalone Task does not justify a new persistent planning-memory scope.
- user-level/global planning preferences remain out of scope.

This mirrors the existing product discipline of explicit, exclusive ownership while avoiding a new global preference subsystem.

---

## 7. PlanningFact Contract

Conceptual shape:

```txt
PlanningFact
- id
- goalId?
- projectId?
- factType
- strength
- structuredValue
- source
- status
- sourcePlanningAttemptId?
- capturedAt
- lastConfirmedAt
- updatedAt
- expiredAt?
- removedAt?
```

Exactly one of `goalId` or `projectId` is present.

### 7.1 Category is derived

`category` is not an independently mutable field.

It is deterministically derived from `factType` using a versioned mapping.

Example:

```txt
UNAVAILABLE_WEEKDAY → AVAILABILITY
AVAILABLE_DEVICE → RESOURCE
CURRENT_LEVEL → STARTING_STATE
LEARNING_FOCUS → PREFERENCE
EXCLUDED_PATH → INTENT
```

A fact cannot be stored with a contradictory category label.

### 7.2 Provenance

Supported provenance includes:

```txt
USER_EXPLICIT
USER_CONFIRMED_AI_EXTRACTION
```

`sourcePlanningAttemptId?` links the fact to the originating Planning attempt/draft when available for audit correlation without preserving the entire conversation as permanent context.

AI-inferred facts that have not been explicitly confirmed do not enter durable planning context.

### 7.3 Initial confirmation timestamp

Approval of a proposed Planning Fact in Draft Review is its first confirmation.

Therefore on initial persistence:

```txt
capturedAt = persistence/capture time
lastConfirmedAt = same initial confirmed time
```

`lastConfirmedAt` is not null for an ACTIVE durable fact.

---

## 8. Fact Status and Lifecycle

Planning Facts use a small lifecycle separate from work-entity lifecycle:

```txt
ACTIVE
EXPIRED
REMOVED
```

### ACTIVE

The fact currently participates in applicable Planning context.

### EXPIRED

A deterministic date-bounded fact has passed its explicit validity range.

Example:

```txt
UNAVAILABLE_DATE_RANGE
2026-12-01 → 2026-12-20
```

after that range ends, the fact may become `EXPIRED` deterministically.

Expiration preserves history and provenance.

### REMOVED

The user explicitly removes the fact from future Planning context.

Removed facts are excluded from active Planning context but may remain in bounded audit history according to accepted retention policy.

AI may suggest that an ACTIVE fact seems stale, but AI may not silently expire, remove, or rewrite a fact whose validity cannot be determined from its structured fields.

---

## 9. Strength Semantics

```txt
HARD
SOFT
INFORMATIONAL
```

### HARD

HARD is permitted only when:

```txt
1. factType is in the closed HARD vocabulary,
2. structuredValue is schema-valid,
3. the current MVP has enough canonical data to deterministically validate violations.
```

A violation in an AI Planning proposal makes that placement invalid.

### SOFT

A user preference the AI should respect but whose violation does not automatically invalidate a proposal.

A materially contradictory proposal may receive a non-blocking warning.

If contradiction detection requires AI interpretation, that warning must be represented as AI-generated review assistance rather than deterministic rule-engine output.

### INFORMATIONAL

Future-relevant context that should inform planning but cannot be generally enforced.

Example:

```txt
AVAILABLE_DEVICE = PHONE_ONLY
```

Without machine-readable Task resource requirements, the system cannot deterministically prove whether every proposed Task can be performed on a phone.

---

## 10. Closed HARD Constraint Vocabulary — MVP

Claude review correctly found that `DAILY_TIME_WINDOW` and `MAX_DAILY_PLANNED_MINUTES` are not deterministically enforceable with the accepted Task model because Task has no canonical planned-time or estimated-duration field.

They are therefore **not HARD types in MVP**.

The initial closed HARD vocabulary is limited to:

```txt
UNAVAILABLE_WEEKDAY
UNAVAILABLE_DATE
UNAVAILABLE_DATE_RANGE
```

### 10.1 UNAVAILABLE_WEEKDAY

```txt
structuredValue:
- weekdays: [MONDAY | TUESDAY | WEDNESDAY | THURSDAY | FRIDAY | SATURDAY | SUNDAY]
```

For Goal or standalone Project scoped AI Task proposals:

```txt
Task.plannedDate local weekday in unavailable set
→ invalid placement
```

When proposing a Routine, the recurrence proposal must not include a prohibited weekday when the recurrence representation can express that constraint.

### 10.2 UNAVAILABLE_DATE

```txt
structuredValue:
- localDate
```

A proposed Task `plannedDate` equal to the unavailable local date is invalid.

### 10.3 UNAVAILABLE_DATE_RANGE

```txt
structuredValue:
- startLocalDate
- endLocalDate
```

A proposed Task `plannedDate` inside the inclusive unavailable local-date range is invalid.

### Deferred candidates

These are not HARD in the current MVP:

```txt
DAILY_TIME_WINDOW
MAX_DAILY_PLANNED_MINUTES
```

They may return only if later accepted canonical Task timing/duration data makes real deterministic enforcement possible.

The UI and AI must not advertise deterministic enforcement for them today.

### Unsupported hard natural-language constraints

If the user expresses a genuinely hard constraint outside the supported vocabulary:

```txt
unsupported HARD meaning
→ visible unsupported-constraint state
→ do not pretend enforcement exists
→ user may rephrase into a supported HARD type
   OR explicitly retain it as SOFT/INFORMATIONAL contextual guidance
```

The product must not silently weaken the user's statement.

Unsupported HARD guidance does not automatically block approval of unrelated work entities, but the limitation must remain visible before approval.

---

## 11. AI Extraction Is Proposal, Not Truth

Extracting durable Planning Facts is an AI responsibility governed by the existing authority model:

```txt
conversation
→ AI proposes structured facts
→ deterministic schema + duplication validation
→ user reviews each durable fact
→ accept / edit / reject
→ accepted facts persist
```

No silent absorption.

No background personality profiling.

No inferred psychological labels.

No persistence merely because information may someday be useful.

### Independent approval units

Approval of work proposals and approval of durable Planning Facts are independent selections inside the same Planning review.

The user may:

```txt
approve valid work entities
while rejecting all proposed Planning Facts
```

or accept some facts and reject others.

Rejecting Planning Facts must not silently delete otherwise valid work proposals.

A rejected fact simply does not become durable context.

---

## 12. Deterministic Enforcement Scope

HARD Planning Facts must not depend only on AI remembering to obey them.

The Discussion 020C pipeline remains the enforcement location:

```txt
provider output
→ schema validation
→ semantic validation
→ temporal validation
→ policy/context validation
→ usable Planning proposal
```

Supported HARD Planning Fact checks participate in those deterministic gates.

Example:

```txt
PlanningFact:
UNAVAILABLE_WEEKDAY = FRIDAY

AI proposes:
Task.plannedDate = Friday

→ deterministic validation failure
```

### Manual user actions

HARD Planning Facts gate AI-generated Planning proposals.

For MVP they do **not** block direct manual user actions such as Quick Create or manual Task/Routine edits.

Rationale:

- a user may intentionally create an exception,
- manual mutation authority belongs to the user,
- forcing all manual paths through a new constraint engine would broaden this discussion substantially.

Known MVP limitation:

```txt
manual edit may contradict an ACTIVE HARD Planning Fact
→ user action remains authoritative
→ no automatic rewrite occurs
```

A later UX may surface a non-blocking conflict warning, but that is not required by Discussion 023 MVP acceptance.

---

## 13. Task and Routine Consumption

Planning Facts are stored once at their applicable Goal or standalone Project scope.

They are not copied onto each Task or Routine.

### Task generation

Every future Planning operation for the scope receives applicable ACTIVE confirmed Planning Facts.

Supported HARD temporal facts are validated against proposed `Task.plannedDate`.

### Routine generation

The same Planning Facts are available while proposing new Routines.

When an accepted HARD fact can be represented directly in `Routine.recurrenceDefinition`, the proposed Routine should encode the compatible recurrence.

After Routine creation, canonical recurrence remains the Routine source of truth.

Do not persist the same recurrence rule again as a Routine-local Planning Fact.

---

## 14. Week-N Planning Context Contract

A future weekly Planning operation receives bounded current context, never an unbounded replay of Planning conversation.

Conceptually:

```txt
WEEKLY_PLANNING_CONTEXT

1. Planning scope
   - Goal OR standalone Project identity

2. Goal / Project current canonical fields
   - desiredOutcome where Goal exists
   - completionMeaning where Project exists
   - targetDate / reviewDate where relevant

3. ACTIVE confirmed applicable Planning Facts
   - HARD constraints
   - SOFT preferences
   - INFORMATIONAL context

4. Active child Projects where Goal-scoped
   - ownership
   - completionMeaning
   - targetDate / reviewDate

5. Active Routines in scope
   - recurrence
   - effective range
   - relevant state

6. Existing relevant Tasks
   - unfinished work
   - placement
   - deadline / review context when relevant

7. Immediately preceding execution-window evidence
   - bounded recent completed Tasks
   - carries/replans where relevant
   - drops where relevant
   - Routine DONE/MISSED facts

8. Relevant deterministic Reconcile facts scoped to entities in this same Goal/Project planning scope

9. Current local date and timezone

10. Next detailed execution horizon
   - maximum seven local calendar days
```

### Bounded history

MVP execution history for rolling planning is limited to the immediately preceding execution window unless a separately accepted deterministic Reconcile fact already summarizes an older pattern.

Do not send unbounded historical Task/Routine logs merely because they exist.

### Scoped Reconcile facts

Reconcile context must be limited to facts whose affected entities belong to the current Goal or standalone Project scope.

Global unrelated Reconcile facts must not leak into the Planning operation.

### Terminal entities

Terminal entities are not included merely for history. If a recent terminal Task/Project is necessary to explain the immediately preceding window, include only the bounded structured evidence needed, not an unbounded terminal history set.

### Context integrity

The context builder must be deterministic, bounded, and versioned.

If required ACTIVE HARD facts are omitted from a Week-N request, Discussion 020C context-integrity validation must reject the operation.

---

## 15. Long-Term Big Picture

The product should preserve long-term direction without persisting a stale free-form AI roadmap as truth.

### Persistent inputs

Use:

```txt
Goal desiredOutcome
Goal targetDate
Projects
Project targetDates / completion meanings
ACTIVE confirmed Planning Facts
actual current lifecycle state
```

### Derived/regenerated presentation

A yearly or multi-month roadmap may be regenerated from current state:

```txt
Goal / Project structure
+
target dates
+
Planning Facts
+
actual progress
+
recent execution / Reconcile evidence
→ current long-term roadmap explanation
```

This is derived AI presentation, not canonical strategy truth.

It must use the existing trust label / authority semantics for AI explanation so the user can distinguish:

```txt
current AI interpretation
≠ previously committed fact
```

Material changes in regenerated roadmap should be presented as an updated interpretation, never as if the user had previously committed to the new narrative.

---

## 16. Reconfirmation Without New Friction

Planning Fact reconfirmation must not turn the accepted low-friction Goal/Project review into a mini configuration session.

### Default behavior

The existing continuation/review interaction remains concise.

When relevant facts may need review, show an optional adjacent entry point such as:

```txt
۳ قید برنامه‌ریزی ثبت شده
بازبینی
```

Do not automatically expand every Planning Fact inside the primary continuation question.

### Goal scope

Use the existing Goal continuation/review surface as the natural optional entry point.

### Standalone Project scope

Use the existing Project review surface analogously.

No new independent recurring timer is introduced solely for Planning Facts.

### Starting-state facts

`STARTING_STATE` facts have high staleness risk.

They should be preferentially surfaced for optional reconfirmation when the parent Goal/Project reaches an existing review checkpoint, but the user should not be forced through a separate mandatory interview.

---

## 17. Staleness and Fact Evolution

Not every fact has the same stability.

Examples:

```txt
more stable:
- explicit exclusions
- durable device/resource constraints

likely to change:
- current skill level
- temporary availability
- date-bounded constraints
```

Rules:

- user edit is authoritative,
- deterministic date-bounded facts may expire from structured dates,
- AI may suggest possible staleness but cannot silently mutate state,
- reconfirmation uses existing Goal/Project review entry points,
- no AI-generated expiration guess becomes truth.

A Starting State fact remains a historical user-confirmed fact until edited/removed; it must not be silently rewritten because execution progress suggests improvement.

---

## 18. Privacy and Data Minimization

Planning Facts may contain sensitive information and inherit accepted privacy, access, minimization, retention, redaction, and observability rules from Discussions 018A and 019C.

Discussion 023 does not create a parallel privacy model.

### Store operational meaning, not unnecessary reason text

For structured constraints, persist only the operational value needed for future planning.

Example:

```txt
User says:
"جمعه‌ها به خاطر مشکل سلامتی نمی‌تونم برنامه داشته باشم."

Persisted HARD fact:
UNAVAILABLE_WEEKDAY = FRIDAY
```

Do **not** persist the unnecessary free-text reason:

```txt
"because of a health condition"
```

unless that additional detail independently satisfies an accepted, necessary product-data purpose, which Discussion 023 does not establish.

Additional rules:

- `structuredValue` stores only the minimum operational value for its factType,
- no arbitrary free-text `reason` field is part of the MVP PlanningFact contract,
- no inferred diagnosis or psychological profile,
- raw conversation is not the durable source of Planning Facts,
- context manifests record scope metadata, counts, and field groups, not sensitive fact values.

### Retention

ACTIVE Planning Facts need durability comparable to the planning scope they serve, not temporary-draft retention.

They inherit the existing user-controlled deletion/access boundaries applicable to durable product state and audit history.

Exact storage-class naming remains an implementation reconciliation task for the 019 family after Discussion 023 is accepted; this open discussion does not invent a competing retention taxonomy.

---

## 19. Provenance and Mutation Authority

Every durable Planning Fact must answer:

```txt
what scope owns it?
what factType is it?
was it explicitly stated or AI-extracted and confirmed?
which Planning attempt proposed it?
when was it captured?
when was it last confirmed?
is it active, expired, or removed?
```

AI is never the authorizing actor for persistence.

```txt
AI extraction
→ proposed fact
→ user confirmation
→ persisted fact
```

Later edits/removals are user-authorized.

Deterministic expiration may only apply where the structured value itself establishes a finite validity range.

---

## 20. PlanningDraft Interaction

PlanningDraft remains ephemeral.

If Discussion 023 is accepted, the PlanningDraft contract may gain:

```txt
proposedPlanningFacts[]
```

These proposals remain unapproved until explicit user confirmation.

Final Planning approval may produce two independent accepted output sets:

```txt
canonical work entities
+
confirmed durable Planning Facts
```

The exact transaction/API shape is deferred to later 019/020 reconciliation.

A PlanningFact proposal overlapping a canonical proposal field must fail the no-duplication gate before final review.

---

## 21. Context Integrity and Observability

If accepted, Planning operations should record Planning Fact context inclusion through the existing `AIContextScopeManifest` model.

Example context category:

```txt
PLANNING_FACTS
```

Manifest records may include:

```txt
scope category
fact count
fact field groups / strength categories
context-builder version
```

They must not duplicate actual sensitive structured values.

A Week-N operation that omits applicable ACTIVE HARD Planning Facts is an invalid-context operation even if the model output parses successfully.

---

## 22. Example — Goal-Scoped Rolling Planning

User:

```txt
I want to reach English B2 within one year.
```

Clarification establishes:

```txt
Current level: A2
Friday: no study
Device: phone only
Preference: conversation-focused
Explicit exclusion: not preparing for IELTS
```

Draft contains:

```txt
Goal proposal
Project / Task / Routine proposals
first seven-day execution
proposed Planning Facts
assumptions / warnings
```

User independently reviews work and durable facts.

Persisted result may include:

```txt
Goal
Projects
Tasks / Routines approved now
confirmed Planning Facts
```

Week 1 executes.

Before Week 2:

```txt
Goal current state
+
ACTIVE confirmed Planning Facts
+
active Projects / Routines / unfinished Tasks
+
immediately preceding execution-window evidence
+
scoped Reconcile facts
+
current local date/timezone
→ Week 2 Planning context
```

If Friday is unavailable:

```txt
Task.plannedDate = Friday
→ deterministic validation failure
```

No original-chat memory is required.

---

## 23. Example — Standalone Project

User starts AI Planning with:

```txt
Plan the next few months for moving apartments.
I cannot do moving-related work on Sundays.
```

No Goal is necessary.

The accepted Planning flow may create a standalone Project.

Its persistent planning context may include:

```txt
PlanningFact
projectId = standalone Project ID
factType = UNAVAILABLE_WEEKDAY
strength = HARD
structuredValue = SUNDAY
```

Future contextual Planning for that Project consumes the same fact without introducing an artificial Goal merely to obtain memory continuity.

---

## 24. Manual Conflict Boundary

Example:

```txt
ACTIVE PlanningFact:
Friday unavailable
```

Later, user manually creates a Task for Friday.

For MVP:

```txt
manual user action is allowed
PlanningFact is not silently changed
Task is not silently moved
```

This is not considered AI constraint-enforcement failure because the user explicitly authored the conflicting action outside AI Planning.

A future non-blocking warning may improve UX, but Discussion 023 does not require a new cross-product manual constraint engine.

---

## 25. Non-Goals and Guardrails

Do not:

- create a canonical Plan entity,
- save the full Planning chat forever as planning truth,
- persist arbitrary AI strategy narratives,
- allow arbitrary AI-invented factType values,
- allow arbitrary AI-invented HARD constraints,
- store independently mutable category when it can be derived from factType,
- duplicate Goal/Project/Task/Routine fields into Planning Facts,
- infer motivation, discipline, personality, diagnosis, or psychological state,
- store unnecessary free-text reasons for operational constraints,
- silently downgrade unsupported HARD meaning,
- silently mutate facts because AI thinks the user's situation changed,
- use stale roadmap narrative as a hidden source of future planning,
- allow Week-N generation with required HARD context silently omitted,
- claim deterministic enforcement where only AI interpretation exists,
- introduce Task time/duration semantics merely to support extra HARD types,
- turn Goal/Project continuation review into a mandatory PlanningFact form.

---

## 26. Claude Review Round 1 — Findings and Resolutions

Claude review round 1 found one Blocking, six Important, and several Minor issues. The direction itself was accepted as coherent subject to these fixes.

### Blocking — unsupported HARD candidates

**Finding:** `DAILY_TIME_WINDOW` and `MAX_DAILY_PLANNED_MINUTES` cannot be deterministically enforced because the accepted Task model has no canonical time-of-day or duration fields.

**Resolution:** ACCEPTED.

They are removed from the MVP HARD vocabulary. MVP HARD types are limited to weekday/date/date-range exclusions.

### Important — category/factType drift

**Resolution:** ACCEPTED.

`category` is derived from `factType`, not independently mutable.

### Important — canonical duplication gate

**Resolution:** ACCEPTED.

A deterministic no-duplication check is added before durable fact review. Existing canonical fields remain authoritative.

### Important — manual edits versus HARD facts

**Resolution:** ACCEPTED WITH MVP BOUNDARY.

HARD facts gate AI Planning proposals. Manual user actions remain authoritative and are not blocked in MVP. A warning may be added later without changing this contract.

### Important — standalone Project continuity

**Resolution:** ACCEPTED.

Planning Facts may belong to exactly one Goal or one standalone Project. Goal-owned Projects normally consume parent Goal facts rather than duplicate them.

### Important — reconfirmation friction

**Resolution:** ACCEPTED.

Planning Fact reconfirmation becomes an optional adjacent entry point on existing Goal/Project review surfaces, not an automatically expanded mini-review.

### Important — unnecessary sensitive reason storage

**Resolution:** ACCEPTED.

MVP Planning Facts store only minimum structured operational values. No arbitrary free-text reason field is added.

### Minor — initial confirmation timestamp

**Resolution:** ACCEPTED.

Initial Draft Review approval sets `lastConfirmedAt`.

### Minor — explicit fact status

**Resolution:** ACCEPTED.

Planning Fact status is `ACTIVE | EXPIRED | REMOVED`.

### Minor — independent approval units

**Resolution:** ACCEPTED.

Work entities and durable Planning Facts are independently selectable during review.

### Minor — provenance linkage

**Resolution:** ACCEPTED.

Optional `sourcePlanningAttemptId` is added.

### Minor — Reconcile scope

**Resolution:** ACCEPTED.

Only Reconcile facts scoped to affected entities inside the same Goal/standalone Project planning scope enter Week-N context.

### Confirmed review guidance integrated

- Planning Facts remain subordinate durable records rather than an embedded free-form blob.
- execution history is bounded to the immediately preceding execution window unless older evidence is already summarized by deterministic Reconcile facts.
- `STARTING_STATE` facts receive priority for optional reconfirmation due to staleness risk.
- fact review should remain compact rather than becoming a separate configuration flow.
- unsupported HARD meaning does not automatically block unrelated work approval.
- context manifests record metadata, not fact values.
- long-term roadmap is derived/regenerated and uses AI explanation trust semantics.

---

## 27. Remaining Questions for Final Claude Re-Review

Round 1 findings are integrated. Final review should now focus only on unresolved contradictions or edge cases introduced by the resolutions.

Please verify:

1. Does Goal-or-standalone-Project exclusive scope create any contradiction with accepted ownership semantics?
2. Is excluding Task/Routine-owned Planning Facts sufficient for MVP, or does a concrete accepted flow still lose necessary rolling context?
3. Is `ACTIVE | EXPIRED | REMOVED` sufficient, especially for facts edited into a new value: update same identity or replace with historical event?
4. Does the canonical no-duplication gate need a closed mapping table from each factType to conflicting canonical fields?
5. For Goal-owned Projects, should all parent Goal Planning Facts automatically apply, or is an applicability subset needed?
6. Can `UNAVAILABLE_DATE` / `UNAVAILABLE_DATE_RANGE` safely expire deterministically using local-date semantics already accepted elsewhere?
7. When a user approves work but rejects all proposed durable facts, is any additional warning required before later rolling Planning loses those facts?
8. Does allowing manual contradiction with HARD facts create any trust contradiction with the word “HARD,” even though enforcement scope is explicitly AI Planning only?
9. Is optional reconfirmation at Goal/Project review sufficient for likely-changing `STARTING_STATE` facts, or is another already-existing signal preferable?
10. Is durable PlanningFact retention sufficiently specified by inheritance, or must a specific existing retention class be chosen before closure?
11. Are there privacy-sensitive factTypes that should be forbidden entirely even when structured and user-confirmed?
12. Does the Week-N context still need an explicit priority/drop order under token pressure, or can it reuse the existing Discussion 020 context-priority contract without restating it here?
13. Is the regenerated long-term roadmap sufficiently grounded without a persisted strategic sequence, given that Project target dates and current structure remain its main sequencing signals?

Review standard remains:

```txt
BLOCKING
IMPORTANT
MINOR
```

For each new finding include:

```txt
Finding
Why it matters
Conflict / edge case
Smallest coherent fix
Affected accepted discussion(s)
```

Do not redesign the product from zero.

---

## 28. Proposed Direction After Round 1

Pending final re-review:

```txt
Planning conversation
→ extract only future-relevant structured facts
→ schema + canonical-duplication validation
→ explicit independent fact review
→ accepted facts persist under Goal OR standalone Project scope

Week N
→ deterministic bounded context builder
→ current canonical scope
   + ACTIVE confirmed Planning Facts
   + immediately preceding execution evidence
   + scoped deterministic Reconcile facts
   + local temporal context
→ AI Planning proposal
→ deterministic supported-HARD validation
→ review / approval
```

Persistent:

```txt
canonical entities
confirmed structured Planning Facts
```

Temporary:

```txt
raw Planning conversation as product-memory source
PlanningDraft after its accepted retention window
unconfirmed extracted facts
```

Derived / regenerated:

```txt
long-term roadmap narrative
yearly glance
current strategy explanation
```

Core product-memory principle:

> The AI should not have to remember the original conversation. The product should remember only validated, user-confirmed, still-applicable facts needed to plan coherently.

---

## 29. Mind Map Impact — NOT YET APPLIED

If Discussion 023 is later accepted, likely Mind Map impact includes:

### Product Model

Potential addition of subordinate durable Planning Facts owned by one Goal or one standalone Project, while preserving the canonical work-entity set.

### AI Responsibilities

Potential additions:

- propose future-relevant Planning Facts from conversation,
- never silently persist extracted facts,
- consume applicable confirmed Planning Facts in future rolling windows,
- distinguish deterministic constraints from preferences/context,
- regenerate long-term roadmap narrative from current facts rather than stale stored strategy text.

### AI Guardrails

Potential additions:

- closed MVP HARD vocabulary,
- no arbitrary AI-invented HARD types,
- no unsupported-HARD silent downgrade,
- no Week-N Planning with applicable HARD context silently omitted,
- no unnecessary sensitive reason persistence,
- no duplicate persistence where a canonical field exists.

### User Flow

Potential addition:

```txt
Planning conversation
→ proposed persistent facts
→ independent fact review
→ approval
→ rolling weekly continuation using durable context
```

### Data / Events / Context

Potential additions:

- PlanningFact provenance/status,
- fact confirmation/edit/removal/expiration events,
- Goal/standalone Project scope ownership,
- context-builder inclusion metadata.

**Do not apply any of these changes while Discussion 023 remains OPEN.**

---

## 30. Affected Formal Documents — NOT YET APPLIED

If accepted, Discussion 023 may require explicit reconciliation or amendment of:

- Discussion 014 family — PlanningDraft output/review contract,
- Discussion 017 family — optional review-surface integration,
- Discussion 019 family — durable persistence, provenance, events, retention,
- Discussion 020 family — context builder, schema, validation, observability,
- Goal/Project review UX specification,
- rolling weekly continuation specification,
- Mind Map and implementation plan.

No formal document is changed by this discussion yet.

---

## 31. Closure Condition

Discussion 023 may be closed only after:

1. final Claude re-review is completed,
2. any remaining Blocking/Important findings are resolved,
3. the final direction is explicitly accepted,
4. affected earlier discussion boundaries are reconciled,
5. Mind Map impact remains recorded but is applied only in a separate explicit step.

Until then:

```txt
STATUS = OPEN
CLAUDE_REVIEW_ROUND_1 = COMPLETED
ROUND_1_FINDINGS = INTEGRATED
FINAL_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

---

# خلاصهٔ فارسی

Discussion 023 هنوز باز است. دور اول review کلاد انجام شده و findingها در نسخهٔ فعلی ادغام شده‌اند، اما هنوز re-review نهایی لازم است و هیچ تغییر این سند در Mind Map یا implementation اعمال نشده است.

مشکل اصلی این Discussion حفظ «حافظهٔ محصول» بین برنامهٔ اولیه و Week 2/3/20 است، بدون اتکا به حافظهٔ session مدل و بدون ذخیرهٔ کل مکالمه. اطلاعاتی که فیلد canonical مناسب دارند همان‌جا می‌روند؛ اطلاعات آینده‌دار بدون خانهٔ canonical می‌توانند به PlanningFact ساختاریافته تبدیل شوند؛ و اطلاعات draft-local بعد از پایان draft دور ریخته می‌شوند.

PlanningFact در MVP متعلق به دقیقاً یک Goal یا یک standalone Project است. category از factType مشتق می‌شود، factها status دارند، provenance و زمان تایید حفظ می‌شود، و تایید آن‌ها مستقل از تایید work entities است.

HARD vocabulary فعلی عمداً فقط شامل `UNAVAILABLE_WEEKDAY`, `UNAVAILABLE_DATE`, و `UNAVAILABLE_DATE_RANGE` است، چون مدل فعلی Task فقط plannedDate دارد و امکان enforcement واقعی برای زمان روز یا سقف دقیقهٔ روزانه وجود ندارد. HARD constraintها AI Planning را gate می‌کنند؛ manual user edits در MVP مسدود نمی‌شوند.

Week-N context از state فعلی، PlanningFactهای ACTIVE، شواهد هفتهٔ بلافصل قبلی، Reconcile facts هم‌scope و زمان محلی ساخته می‌شود. yearly glance و roadmap به‌صورت derived AI explanation از state زنده بازتولید می‌شوند و narrative قدیمی به‌عنوان حقیقت دائمی ذخیره نمی‌شود.

PlanningFactها فقط minimum structured operational value را نگه می‌دارند و دلیل‌های حساس و غیرضروری مثل علت پزشکی یک محدودیت ذخیره نمی‌شوند. reconfirmation نیز به شکل entry point اختیاری کنار Goal/Project review موجود انجام می‌شود، نه یک چرخهٔ مزاحم جدید.
