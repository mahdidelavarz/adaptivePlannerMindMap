# Discussion 023 — Persistent Planning Facts and Rolling Execution Context

## Status

**OPEN — proposed direction, pending Claude review.**

This discussion is **not yet accepted or closed**.

Nothing in this file is authoritative product behavior until the review is completed and the resulting decisions are explicitly accepted.

**Mind Map status: NOT APPLIED.**

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

A user may request a one-year plan, while the AI creates only the first seven days of detailed Task and Routine execution.

Discussion 014 already defines:

```txt
Week 1
→ calibration period
→ explicit first continuation decision
→ later signal-based continuation
```

However, a missing contract remains:

> When Week 2, Week 3, or Week 20 is generated, what durable product context preserves the important facts the user gave during the original planning conversation?

Example:

```txt
User goal:
Reach English B2 within one year.

Clarification answers:
- current level is A2
- about 45 minutes are available per day
- Friday must remain completely free from study
- only a phone is available
- conversation is more important than grammar-heavy study
- IELTS preparation is not part of the goal
```

The first PlanningDraft may use all of this information correctly.

After approval, the raw conversation or temporary PlanningDraft must not become the product's permanent memory by accident.

But if none of the future-relevant information survives, Week 2 may know only:

```txt
Goal title
Goal desiredOutcome
Projects
current Tasks and Routines
```

and lose important constraints, resources, preferences, baseline state, or explicit exclusions.

The product must not depend on model-session memory for correctness.

Core requirement:

> Weekly Planning AI must receive the durable planning context it needs from product state, not from an assumption that the model remembers an earlier conversation.

---

## 2. Scope

This discussion proposes rules for:

1. which information from Planning conversation may survive approval,
2. which information must instead be written into existing canonical entity fields,
3. which information must remain temporary and disappear with the draft,
4. a structured `PlanningFact` model for future-relevant user facts that have no existing canonical home,
5. HARD, SOFT, and INFORMATIONAL semantics,
6. a closed vocabulary for deterministically enforceable HARD constraints,
7. AI extraction and explicit user review of proposed Planning Facts,
8. editing, deleting, and reconfirming persisted facts,
9. deterministic validation of supported HARD constraints,
10. assembly of context for Week N rolling execution planning,
11. regeneration of long-term roadmap narratives instead of persisting stale AI strategy text,
12. privacy, retention, provenance, and observability inheritance.

### Explicitly out of scope

This discussion does not propose:

- a canonical `Plan` entity,
- persistence of the full AI conversation as planning truth,
- persistence of arbitrary AI strategy narratives,
- a global user planning-profile system,
- a universal constraint language,
- deterministic enforcement of every natural-language preference,
- a new Reconcile severity model,
- a new Goal lifecycle,
- detailed production database/API implementation before the product contract is accepted,
- Mind Map changes while this discussion is open.

---

## 3. Existing Accepted Constraints That Must Remain Intact

Discussion 023 must not silently replace these accepted decisions.

### 3.1 No canonical Plan entity

The canonical MVP entities remain:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

`PlanningDraft` may be temporarily persisted for recovery and approval workflows but is not canonical work truth.

### 3.2 Seven-day detailed execution horizon

Long-lived Goals, Projects, and Routines may span months or years.

Detailed Task execution generated by one PlanningDraft remains limited to no more than seven consecutive local calendar days.

Discussion 023 does not authorize a 365-day scheduled Task backlog.

### 3.3 Approval boundary

AI proposes.

The user approves.

No proposed durable Planning Fact may silently become trusted product state merely because the model inferred or extracted it.

### 3.4 Derive instead of duplicate

Existing source-of-truth discipline remains mandatory.

Examples already accepted elsewhere include:

```txt
FirstWeekEntry.date
→ derived from Task.plannedDate

nextTemporalCheckpoint
→ derived

carryCount
→ derived from events
```

Planning Facts must not duplicate information that already has a correct canonical home.

---

## 4. Proposed Governing Principle

Do not persist the planning conversation.

Do not persist AI strategy narrative as long-term truth.

Persist only reviewed, future-relevant, structured user planning facts that:

```txt
1. materially affect future planning,
2. do not already have an appropriate canonical field,
3. can be represented without inventing psychological or behavioral conclusions,
4. have explicit provenance,
5. are visible and editable by the user.
```

Then rebuild every later weekly planning context from:

```txt
canonical current state
+
confirmed Planning Facts
+
recent execution evidence
+
relevant deterministic Reconcile facts
+
current local temporal context
```

Model memory is never a correctness dependency.

---

## 5. Three-Way Information Placement Rule

Every piece of information obtained during Planning clarification should be classified using this rule.

### Case A — Existing canonical home exists

Store it only in that canonical field.

Examples:

```txt
user specifies Goal target date
→ Goal.targetDate

user specifies Task execution date
→ Task.plannedDate

user defines a recurring weekday pattern
→ Routine.recurrenceDefinition

user defines Project completion meaning
→ Project.completionMeaning
```

Do not also duplicate the same fact inside Goal Planning Context.

### Case B — Future-relevant but no canonical home exists

Propose a structured Planning Fact for explicit review.

Examples may include:

```txt
current English level is A2
Friday must remain unavailable for this Goal
only a phone is available
user generally has 45 minutes per study day
conversation-focused learning is preferred
IELTS preparation is explicitly excluded
```

### Case C — Draft-local only

Keep it only inside the Planning flow and discard it when the draft lifecycle ends.

Example:

A clarification answer that exists only to choose between two draft structures and has no later planning value should not become durable merely because it appeared in conversation.

Core rule:

> Persistence is justified by future planning value, not by the fact that something was said to AI.

---

## 6. Proposed Goal-Scoped Planning Context

For MVP, durable Planning Facts are proposed to be scoped to a Goal.

Conceptually:

```txt
Goal
└── PlanningContext
    └── PlanningFact[]
```

This does **not** create a new canonical work entity comparable to Goal, Project, Task, or Routine.

The exact persistence shape remains open for review.

The product-level concept is:

> Goal-scoped durable planning context containing user-reviewed facts needed to generate future execution windows coherently.

### Why Goal-level first

The current identified problem is continuity of planning for one long-running Goal.

A global profile such as:

```txt
Never schedule anything on Friday across my entire life.
```

may eventually justify user-level planning preferences, but that is not required to resolve the current Goal continuity gap.

MVP should not introduce a global planning-preference system without evidence.

---

## 7. Proposed PlanningFact Contract

Conceptual shape:

```txt
PlanningFact
- id
- goalId
- category
- factType
- strength
- structuredValue
- source
- capturedAt
- lastConfirmedAt?
- updatedAt
```

Potential provenance:

```txt
source:
USER_EXPLICIT
USER_CONFIRMED_AI_EXTRACTION
```

AI-inferred facts that have not been explicitly confirmed must not enter durable Goal planning context.

### Proposed category vocabulary

```txt
CONSTRAINT
AVAILABILITY
RESOURCE
PREFERENCE
STARTING_STATE
INTENT
```

`category` describes planning meaning.

`factType` describes the specific supported fact shape.

The exact final taxonomy is open for Claude review.

---

## 8. Strength Semantics

Proposed strengths:

```txt
HARD
SOFT
INFORMATIONAL
```

### HARD

A fact is HARD only when the product has a known machine-readable representation and a deterministic enforcement rule.

Examples:

```txt
UNAVAILABLE_WEEKDAY
UNAVAILABLE_DATE
UNAVAILABLE_DATE_RANGE
```

A HARD fact may block a proposed detailed execution placement that violates it.

### SOFT

A user preference that Planning AI should respect but whose violation does not automatically make the proposal invalid.

Example:

```txt
Prefer conversation practice over grammar-heavy study.
```

A visible non-blocking warning may be appropriate when a draft materially contradicts a confirmed SOFT preference.

### INFORMATIONAL

A future-relevant fact that should inform AI reasoning but cannot be deterministically enforced as a general rule.

Example:

```txt
Available device: phone only.
```

The system may not always know deterministically whether a proposed Task requires a laptop.

Therefore this fact belongs in Planning context without pretending that a deterministic gate exists for every possible violation.

---

## 9. Closed HARD Constraint Vocabulary

The HARD vocabulary must be closed and versioned.

AI must never invent a new HARD `factType` at runtime.

A deterministic validator must know exactly what each supported HARD type means and how to validate it.

Initial candidate MVP vocabulary for review:

```txt
UNAVAILABLE_WEEKDAY
UNAVAILABLE_DATE
UNAVAILABLE_DATE_RANGE
DAILY_TIME_WINDOW
MAX_DAILY_PLANNED_MINUTES
```

Potential structured shapes:

```txt
UNAVAILABLE_WEEKDAY
- weekdays: [MONDAY | TUESDAY | WEDNESDAY | THURSDAY | FRIDAY | SATURDAY | SUNDAY]
```

```txt
UNAVAILABLE_DATE
- localDate
```

```txt
UNAVAILABLE_DATE_RANGE
- startLocalDate
- endLocalDate
```

```txt
DAILY_TIME_WINDOW
- startLocalTime
- endLocalTime
```

```txt
MAX_DAILY_PLANNED_MINUTES
- minutes
```

The final set must be reviewed for overlap, timezone/local-date semantics, Task fields actually available in MVP, and whether each candidate can truly be enforced with current product data.

### Important downgrade rule

If a user expresses a hard natural-language constraint that is not representable by the supported closed HARD vocabulary, the system must not silently pretend deterministic enforcement exists.

It must not silently convert the user's meaning into a weaker constraint either.

Instead, the draft review should make the limitation visible.

Conceptually:

```txt
unsupported hard natural-language constraint
→ visible unsupported-constraint state/warning
→ user may edit/rephrase into a supported structured constraint
   OR explicitly keep it as contextual SOFT/INFORMATIONAL guidance
```

The product must not weaken a user statement without visibility.

---

## 10. AI Extraction Is a Proposal, Not Truth

Extracting future-relevant Planning Facts from conversation is a new AI responsibility.

It must follow the existing authority model:

```txt
conversation
→ AI proposes structured facts
→ deterministic schema validation
→ user reviews
→ user accepts / edits / rejects
→ accepted facts become durable context
```

No silent absorption.

No background personality profiling.

No inferred psychological labels.

No conversion of ordinary conversational content into durable user facts merely because it might someday be useful.

### Draft Review requirement

Planning review should show proposed persistent facts distinctly enough that the user understands:

> These details will be remembered for future planning under this Goal.

The user must be able to:

```txt
accept
edit
reject
```

individual proposed facts.

Final UX treatment remains out of scope until the contract is accepted.

---

## 11. Deterministic Enforcement

HARD Planning Facts must not depend only on AI remembering to obey them.

The accepted Discussion 020C validation pipeline already establishes that AI output becomes usable only after semantic, temporal, reference, policy, and context validation gates succeed.

Discussion 023 proposes adding supported HARD Planning Fact checks into the appropriate deterministic validation stages.

Example:

```txt
Goal Planning Fact:
UNAVAILABLE_WEEKDAY = FRIDAY

AI proposes Task:
plannedDate = Friday

→ deterministic validation failure
→ proposal cannot become a valid reviewable execution placement in that form
```

The system, not only the model, protects the accepted constraint.

### No false deterministic claims

Some Planning Facts cannot be generally validated deterministically.

Example:

```txt
AVAILABLE_DEVICE = PHONE_ONLY
```

Without machine-readable Task resource requirements, the validator cannot prove whether an arbitrary proposed Task requires a laptop.

Such a fact may inform AI reasoning but must not be advertised as deterministically enforced.

---

## 12. Routine and Task Consumption

A Goal-level Planning Fact is stored once.

It is not copied onto every Task or Routine.

### Task generation

Every future Planning operation for the Goal must receive applicable confirmed Planning Facts.

Supported HARD temporal constraints are validated against proposed Task placement.

Example:

```txt
UNAVAILABLE_WEEKDAY = FRIDAY
→ Week 2 Planning must not place Goal-related Tasks on Friday
```

### Routine generation

The same Goal Planning Facts are available while proposing new Routines.

When an accepted HARD constraint can be represented directly by the Routine recurrence definition, the resulting Routine should encode the canonical recurrence correctly.

After creation, canonical recurrence remains the source of truth for that Routine.

Do not duplicate the same schedule rule into another Routine-local narrative field.

---

## 13. Proposed Week-N Planning Context Contract

The missing continuation contract should be explicit.

A future weekly Planning operation for an existing Goal should receive a bounded context assembled from current product truth.

Conceptually:

```txt
WEEKLY_PLANNING_CONTEXT

1. Goal
   - desiredOutcome
   - targetDate
   - reviewDate where relevant

2. Confirmed Goal Planning Facts
   - supported hard constraints
   - availability
   - resources
   - preferences
   - starting-state facts still relevant
   - explicit exclusions / intent facts

3. Active Projects
   - ownership
   - status
   - completionMeaning
   - targetDate / reviewDate where relevant

4. Active Routines
   - recurrence
   - effective range
   - relevant current state

5. Relevant existing Tasks
   - unfinished work
   - current placement
   - deadline / review context when relevant

6. Recent execution evidence
   - completed Tasks
   - carried/replanned Tasks where relevant
   - dropped Tasks where relevant
   - Routine DONE/MISSED facts

7. Relevant deterministic Reconcile facts/signals

8. Current local date and timezone

9. Next detailed execution horizon
   - maximum seven local calendar days
```

The exact DTO, field limits, history window, and token-budget rules belong to later technical refinement after product acceptance.

### Critical invariant

The context builder must be deterministic and bounded.

Missing model memory must never cause context widening or improvisational reconstruction of the original conversation.

---

## 14. Long-Term Big Picture

The original gap also raises a separate question: how should the user retain a useful one-year or multi-month big-picture view?

Proposed direction:

### Persist facts and canonical structure

Persist:

```txt
Goal desiredOutcome
Goal targetDate
Projects
Project targetDates / completion meanings where applicable
confirmed Goal Planning Facts
actual current lifecycle and execution state
```

### Do not persist stale AI roadmap narrative as canonical truth

Avoid storing a free-form block such as:

```txt
Months 1–3: fundamentals
Months 4–6: architecture
Months 7–9: portfolio
...
```

as an indefinitely trusted planning source unless a future discussion defines explicit ownership, revision, and invalidation semantics.

### Regenerate the view

A user-facing long-term roadmap or yearly glance may be generated from current facts:

```txt
Goal
+
Projects and targetDates
+
confirmed Planning Facts
+
actual progress
+
recent execution / Reconcile evidence
→ fresh long-term roadmap explanation
```

Such narrative is derived AI presentation unless explicitly approved into an existing canonical field.

This preserves the big picture without creating a stale parallel strategy source.

---

## 15. Update, Delete, and Reconfirmation

Persisted Planning Facts cannot be write-once memory.

The user must be able to edit or remove them.

### Proposed reconfirmation approach

Do not introduce a new independent recurring interruption merely for Planning Facts.

Use an existing Goal continuation/review interaction when appropriate to surface relevant facts for lightweight reconfirmation.

Example:

```txt
Goal review
→ Are these planning constraints still accurate?
→ unchanged / edit / remove
```

Exact cadence must inherit existing Goal review/continuation policy rather than inventing a competing timer.

### Important open point

The repository contains accepted Goal review and continuation mechanisms, but the precise UX and timing for fact reconfirmation must be checked against those accepted rules during review.

Discussion 023 must not casually introduce a new fixed 14–30 day mechanism if that interval is not already authoritative.

---

## 16. Staleness Rules

Not every fact has the same staleness risk.

Potential future policy may distinguish:

```txt
stable facts
- explicit goal exclusion
- durable resource limitation

likely-changing facts
- available minutes
- unavailable date range
- current starting level
```

However, the MVP should avoid inventing AI-generated expiration guesses.

Until a more specific rule is accepted:

- user edits are authoritative,
- explicit Goal review may reconfirm facts,
- deterministic date-bounded facts naturally expire when their date range passes,
- AI may suggest that a fact appears stale but may not silently mutate or delete it.

---

## 17. SOFT Constraint / Preference Violations

A SOFT or INFORMATIONAL fact does not create a deterministic blocking gate merely because AI may have ignored it.

However, material contradiction should not necessarily remain invisible.

Proposed direction:

```txt
confirmed SOFT preference
+
materially contradictory proposal
→ non-blocking visible warning when reliably detectable
```

Important limitation:

If contradiction detection itself requires open-ended AI interpretation, the product must represent the result as AI-generated review assistance, not deterministic fact.

Do not falsely label AI interpretation as rule-engine enforcement.

---

## 18. Privacy, Sensitive Data, and Retention

Planning Facts may contain sensitive information.

Example:

```txt
A health-related limitation affects scheduling.
```

Discussion 023 must not invent a parallel privacy model.

Proposed rule:

Planning Facts inherit the strictest applicable data-minimization, access, retention, raw-content, and observability requirements from the accepted AI/privacy/persistence discussions, especially Discussions 018A and 019C.

Additional principles:

- store only the structured future-relevant fact needed for planning,
- do not preserve unnecessary raw conversational wording,
- do not create psychological profiles,
- do not store inferred diagnoses,
- context manifests should record scope metadata rather than duplicate fact content,
- raw AI prompt/response retention does not become the source of Planning Facts.

The exact retention class for durable Goal Planning Facts remains open for review because they differ from temporary PlanningDraft content: they may need to live as long as the Goal while still respecting user deletion and sensitive-data controls.

---

## 19. Provenance and Mutation Authority

Every durable Planning Fact must preserve enough provenance to answer:

```txt
Was this explicitly stated by the user?
Was it extracted by AI and then confirmed?
When was it captured?
When was it last confirmed?
```

AI is never the authorizing actor for persistence.

Conceptually:

```txt
AI extraction
→ proposed fact
→ user acceptance
→ persisted fact
```

Later edits and removals are user-authorized changes.

If deterministic system behavior marks a date-bounded fact inactive after its explicit range has passed, that must be defined separately and must not rewrite historical provenance.

---

## 20. PlanningDraft Interaction

PlanningDraft remains ephemeral.

A future version of the draft contract may include:

```txt
proposedPlanningFacts[]
```

for review before approval.

These are not durable merely because they exist inside the draft.

Final approval may conceptually produce two categories of accepted output:

```txt
canonical work entities
+
confirmed durable Goal Planning Facts
```

This does not make Planning Facts a `Plan` entity.

The exact transaction/API shape is deferred until this discussion is accepted and the affected 019/020 contracts are amended coherently.

---

## 21. Context Integrity and Observability

Discussion 019C already defines `AIContextScopeManifest` so the product can record which logical context categories were included without duplicating raw content.

If Planning Facts are accepted, Planning operations should eventually record that Goal Planning Fact context was included through context-scope metadata.

Example category:

```txt
GOAL_PLANNING_FACTS
```

The manifest should record scope/category inclusion, count, and applicable field-group metadata, not duplicate the sensitive values themselves.

Context completeness must participate in the existing Discussion 020C context-integrity gate.

A Week-N proposal generated after silently omitting required confirmed HARD constraints should be rejected as invalid context use, not accepted because the output happens to parse.

---

## 22. Example End-to-End Flow

User enters:

```txt
I want to reach English B2 within one year.
```

AI clarification establishes:

```txt
Current level: A2
Available time: about 45 minutes per study day
Friday: no study
Device: phone only
Preference: conversation-focused
Explicit exclusion: not preparing for IELTS
```

AI produces a PlanningDraft containing:

```txt
Goal proposal
Project / Task / Routine proposals
first seven-day detailed execution
proposed durable Planning Facts
visible assumptions / warnings
```

User reviews and approves selected content.

Persisted result may include:

```txt
Goal
Projects
Tasks / Routines approved now
confirmed Goal Planning Facts
```

Week 1 executes.

Before Week 2 generation, the product assembles:

```txt
Goal current state
confirmed Planning Facts
active Projects
active Routines
relevant unfinished Tasks
Week 1 execution evidence
Reconcile facts
current local date/timezone
```

AI proposes Week 2.

Deterministic validation checks supported HARD constraints.

Example:

```txt
Friday is unavailable
→ Friday Task placement is invalid
```

The resulting Week 2 proposal is therefore continuous with the user's original planning intent without requiring persistence of the full original conversation or reliance on model-session memory.

---

## 23. Proposed Non-Goals and Guardrails

Do not:

- create a canonical Plan entity,
- save the entire Planning chat forever as planning truth,
- allow arbitrary AI-invented Planning Fact types,
- allow arbitrary AI-invented HARD rules,
- duplicate Goal/Project/Task/Routine fields into Planning Facts,
- infer motivation, discipline, personality, health diagnosis, or psychological state,
- silently downgrade unsupported HARD user constraints,
- silently mutate facts because AI thinks the user's situation changed,
- use stale long-term AI narrative as a hidden source of future weekly planning,
- allow Week-N generation without required Goal context,
- claim deterministic enforcement where only AI interpretation exists.

---

## 24. Open Questions for Claude Review

Claude should review the proposed direction specifically for contradictions, missing states, ambiguity, edge cases, and conflicts with accepted Discussions 012–022.

Please do **not** redesign the product from zero or reject the direction merely to reduce MVP size.

Review these questions in particular:

### Data model and semantics

1. Should Goal Planning Facts be modelled as subordinate durable records, embedded structured fields, or another persistence shape while remaining outside the canonical work-entity set?
2. Is `PlanningFact.category + factType + strength + structuredValue` sufficiently explicit, or does it permit hidden semantic ambiguity?
3. Which fields are necessary for provenance and reconfirmation without overbuilding history?
4. Should `lastConfirmedAt` be nullable and what exactly constitutes confirmation?

### HARD vocabulary

5. Which candidate HARD fact types are truly enforceable with the accepted MVP Task/Routine fields?
6. Is `DAILY_TIME_WINDOW` enforceable if Tasks do not always carry an execution time?
7. Is `MAX_DAILY_PLANNED_MINUTES` enforceable if Tasks do not always have duration data?
8. Should unsupported hard natural-language constraints block approval, remain visible warnings, or require explicit user downgrade to contextual guidance?

### Duplication and source of truth

9. How should the system detect when a proposed Planning Fact actually belongs in an existing canonical field?
10. What prevents duplication between Goal Planning Facts and Routine recurrence / Project target dates / Task deadlines?
11. When a canonical field later changes, can a previously related Planning Fact become contradictory and how should that be surfaced?

### Rolling context

12. What is the minimum bounded Week-N context required to preserve continuity without token bloat?
13. Which execution-history window is appropriate?
14. Which Reconcile facts are relevant enough to include?
15. Should Week-N context include terminal Projects/Tasks for recent history, and if so under what bounded rule?
16. What happens when no Goal exists because the accepted Planning flow also supports standalone Project/Task/Routine intentions?

### Fact lifecycle

17. How should date-bounded facts expire without disappearing from historical audit?
18. How should Starting State facts evolve? Example: `English level = A2` becomes stale after months of progress.
19. Which facts should be reconfirmed during Goal review and which need no repeated prompt?
20. Does fact reconfirmation conflict with the accepted low-friction Goal review model?

### UX / authority

21. How should proposed durable facts be presented during draft review without turning Planning into a long configuration form?
22. Can a user approve work entities while rejecting all proposed durable facts?
23. Should an unsupported HARD constraint block work approval if future planning cannot guarantee it?

### Privacy and persistence

24. Which accepted retention/access class should durable Planning Facts inherit?
25. How should sensitive fact values be exposed to AI context builders while respecting the existing scope manifest and raw-content boundaries?
26. Are there fact categories that should never be persisted even with user confirmation?

### Derived big picture

27. Is deriving/regenerating a long-term roadmap from Goal + Projects + target dates + Planning Facts + actual progress sufficient, or is some explicit user-approved strategic sequencing still missing?
28. If generated roadmap narrative changes materially between reviews, how should the product communicate that it is an updated interpretation rather than previously committed truth?

---

## 25. Proposed Review Standard

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

- contradiction with accepted product model,
- source-of-truth duplication,
- stale-state risk,
- enforcement honesty,
- authority boundaries,
- privacy implications,
- missing lifecycle states,
- rolling-context completeness,
- low-friction UX.

Do not optimize for implementation convenience at the expense of product coherence.

---

## 26. Current Proposed Direction Summary

Pending review, the proposed direction is:

```txt
Planning conversation
→ extract only future-relevant structured facts
→ deterministic schema validation
→ explicit user review
→ accepted Goal-scoped Planning Facts persist

Week N
→ deterministic bounded context builder
→ canonical current state
   + confirmed Planning Facts
   + recent execution evidence
   + relevant Reconcile facts
   + local temporal context
→ AI Planning proposal
→ deterministic HARD-constraint validation
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
```

Derived / regenerated:

```txt
long-term strategy narrative
yearly glance
current roadmap explanation
```

Core product-memory principle:

> The AI should not have to remember the original conversation. The product should remember only the validated, still-relevant facts needed to plan coherently.

---

## 27. Mind Map Impact — NOT YET APPLIED

If Discussion 023 is later accepted, likely Mind Map areas affected include:

### Product Model

Potential addition of Goal-scoped persistent Planning Facts as subordinate planning context, while preserving the canonical work-entity set.

### AI Responsibilities

Potential additions:

- propose future-relevant Planning Facts from conversation,
- never silently persist extracted facts,
- consume confirmed Goal Planning Facts in future weekly windows,
- distinguish deterministic constraints from preferences and context,
- generate long-term roadmap narrative from current facts rather than stale stored strategy text.

### AI Guardrails

Potential additions:

- closed HARD fact vocabulary,
- no arbitrary AI-invented HARD constraints,
- no silent unsupported-HARD downgrade,
- no Week-N generation with incomplete required context,
- no psychological inference as persistent planning fact,
- no duplicate persistence where a canonical field exists.

### User Flow

Potential additions:

```txt
Planning conversation
→ proposed persistent facts
→ user review
→ approval
→ rolling weekly continuation using durable context
```

### Data / Events / Context

Potential additions:

- planning-fact provenance,
- fact confirmation/edit/removal events,
- context-builder inclusion of Goal Planning Facts,
- context-scope manifest category for Planning Facts.

**Do not apply any of these changes while Discussion 023 remains OPEN.**

---

## 28. Affected Formal Documents — NOT YET APPLIED

If accepted, Discussion 023 may require explicit amendments to:

- PlanningDraft product contract (Discussion 014 family),
- canonical persistence model / invariants (Discussion 019 family),
- AI context builder and operation contracts (Discussion 020 family),
- validation gates for HARD planning constraints,
- Planning privacy/retention documentation,
- Goal review UX specification,
- rolling weekly continuation specification,
- Mind Map and implementation plan.

No formal document is changed by this discussion yet.

---

## 29. Closure Condition

Discussion 023 may be closed only after:

1. Claude review is completed,
2. blocking and important findings are resolved,
3. the final product direction is explicitly accepted,
4. affected earlier discussion boundaries are reconciled,
5. the Mind Map impact is recorded for a separate application step.

Until then:

```txt
STATUS = OPEN
CLAUDE_REVIEW = PENDING
MIND_MAP = NOT_APPLIED
IMPLEMENTATION = NOT_AUTHORIZED_FROM_THIS_FILE
```

---

# خلاصهٔ فارسی

Discussion 023 شکاف بین «برنامه‌ریزی اولیه» و «ادامهٔ هفتگی» را بررسی می‌کند. مدل فعلی اجازه می‌دهد کاربر یک Goal یک‌ساله تعریف کند ولی AI فقط هفت روز اجرای دقیق پیشنهاد دهد. مشکل این است که اطلاعات مهمی که کاربر در مکالمهٔ اولیه گفته و خانهٔ canonical مشخصی ندارند، ممکن است تا هفته‌های بعد گم شوند.

جهت پیشنهادی این است که کل مکالمه یا narrative استراتژی AI ذخیره نشود. فقط factهای آینده‌دار، ساختاریافته و تأییدشدهٔ کاربر در سطح Goal نگه داشته شوند. اگر اطلاعات از قبل فیلد canonical مناسب داشته باشد، همان فیلد منبع حقیقت می‌ماند و Planning Fact ساخته نمی‌شود.

Planning Factها به سه سطح HARD، SOFT و INFORMATIONAL تقسیم می‌شوند. HARD فقط برای vocabulary بسته‌ای مجاز است که سیستم واقعاً می‌تواند آن را deterministic enforce کند. AI حق ندارد نوع HARD جدید اختراع کند یا محدودیت سخت پشتیبانی‌نشده را بی‌صدا ضعیف کند. استخراج facts توسط AI فقط proposal است و کاربر باید آن‌ها را ببیند، ویرایش کند، رد کند یا تأیید کند.

برای ساخت Week 2، Week 3 و هفته‌های بعد، سیستم باید یک context محدود و deterministic از Goal فعلی، Planning Facts تأییدشده، Projects/Routines/Tasks مرتبط، شواهد اخیر اجرا، Reconcile facts و زمان محلی بسازد. صحت برنامه نباید به حافظهٔ session مدل وابسته باشد.

نمای بلندمدت یا yearly glance بهتر است از state واقعی دوباره تولید شود و به‌عنوان narrative قدیمی و دائمی ذخیره نشود. این Discussion هنوز باز است، برای review کلاد نوشته شده، در Mind Map اعمال نشده و تا زمان بسته‌شدن مبنای implementation نهایی نیست.
