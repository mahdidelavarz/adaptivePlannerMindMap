# Discussion 014 — AI Planning Output Contract

## Status

Open for GPT × Claude review.

This version contains GPT's proposed decisions and is not yet accepted.

Related accepted discussions:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]

---

## 1. Scope

This discussion defines the structured, bounded planning draft produced by Planning AI before user approval.

It answers:

```txt
What may the draft contain?
How does it map to Goal, Project, Task, and Routine?
What limits prevent oversized or incoherent drafts?
How are assumptions, warnings, invalid values, and partial results represented?
What may the user edit and approve?
```

The draft is ephemeral. It is not a canonical product entity and creates nothing by itself.

Core invariant inherited from Discussion 013:

> Conversation produces a proposal. Approval creates product entities.

---

## 2. Explicitly Out of Scope

This discussion does not define:

- conversation entry or clarification flow — Discussion 013
- persistence tables, database constraints, transactions, or events — Discussion 019
- model provider, runtime, retries, schema-enforcement library, or API transport — Discussion 020
- Today execution, occurrence generation, or Carry behavior — Discussion 015
- Reconcile output or adaptation actions — Discussions 016 and 017
- detailed safety detection or provider failure policy — Discussion 018
- implementation sequencing — Discussion 022

A structured contract is defined here conceptually. Its eventual JSON Schema, DTOs, database mapping, and API envelope belong to later technical discussions.

---

## 3. Accepted Dependencies

Discussion 012 established:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

The planning draft may propose creation of:

```txt
Goal
Project
Task
Routine
```

It does not directly propose `RoutineOccurrence`. Occurrences belong to scheduling and execution behavior.

Ownership constraints remain unchanged:

```txt
Project may belong to one Goal or be standalone.
Task may belong to one Goal, one Project, or neither.
Routine may belong to one Goal, one Project, or neither.
Task and Routine may not belong to Goal and Project simultaneously.
```

Discussion 013 established:

- broad and narrow intentions are supported
- Goal creation is not mandatory
- clarification may be skipped
- the user may request `Draft now`
- assumptions must be visible
- no canonical entities exist before approval
- safety fallback produces no planning draft

---

## 4. Proposed Contract Principles

### 4.1 Draft, Not Hidden Execution

Planning AI returns a proposal that the product can render and validate.

It must not:

- create canonical entities before approval
- return hidden actions that bypass review
- invent product concepts outside Discussion 012
- silently repair a materially different user intention
- claim that a proposed date, recurrence, or ownership has already been applied

### 4.2 Product Meaning Before Transport Shape

The contract describes product meaning first.

Field names shown below are canonical conceptual names for this discussion, but exact API casing, serialization details, IDs, and DTO structure belong to Discussions 019 and 020.

### 4.3 Minimal Complete Draft

A draft should contain only the structures justified by the user intention.

Valid examples include:

```txt
one standalone Task
one standalone Routine
one standalone Project with Tasks
one Goal with direct Tasks and Routines
one Goal with Projects and child work
multiple standalone Tasks
```

A draft does not become better merely because it contains more hierarchy, a recurring software superstition with impressive endurance.

---

## 5. Proposed PlanningDraft Shape

```txt
PlanningDraft
- draftSummary
- proposals[]
- assumptions[]
- warnings[]
- unresolvedQuestions[]
- suggestedFirstWeek?
```

### 5.1 `draftSummary`

A short user-facing summary of what the AI understood and proposed.

Rules:

- one to three short sentences
- no unsupported claim that the Goal is achievable
- no motivational diagnosis or personality inference
- no hidden decision absent from the structured proposals

Example:

> I interpreted this as a Goal to find a frontend job, supported by one portfolio Project, several one-off Tasks, and a recurring interview-practice Routine.

### 5.2 `proposals[]`

An ordered collection of proposed canonical entities.

Each proposal contains:

```txt
Proposal
- draftId
- entityType: GOAL | PROJECT | TASK | ROUTINE
- title
- description?
- parentDraftId?
- source: EXPLICIT | INFERRED
- confidence: HIGH | MEDIUM | LOW
- fields
```

#### `draftId`

A temporary identifier used only to connect items inside the draft.

It is not a canonical product ID.

#### `entityType`

Must be one of the four entities that Planning AI may propose:

```txt
GOAL
PROJECT
TASK
ROUTINE
```

#### `title`

Required, concise, and user-editable.

#### `description`

Optional supporting detail. It must not duplicate the title merely to satisfy a field.

#### `parentDraftId`

Optional reference to another proposal in the same draft.

Allowed parent mappings:

```txt
PROJECT → GOAL
TASK → GOAL | PROJECT
ROUTINE → GOAL | PROJECT
```

Forbidden mappings:

```txt
GOAL → any parent
PROJECT → PROJECT
TASK → TASK | ROUTINE
ROUTINE → TASK | ROUTINE
```

A Task or Routine with no `parentDraftId` is standalone.

#### `source`

```txt
EXPLICIT
```

The proposal directly reflects user-provided intent.

```txt
INFERRED
```

The AI added or structured the proposal beyond the user's literal wording.

This label supports transparent review. It is not evidence that inferred content is correct.

#### `confidence`

Confidence describes interpretation confidence, not predicted success.

```txt
HIGH
The item directly follows from explicit user input.

MEDIUM
The item is a reasonable interpretation with limited ambiguity.

LOW
The item depends on a visible assumption or may require user correction.
```

The system must not present confidence as a psychological or statistical score.

### 5.3 Entity-Specific Fields

#### Goal Fields

```txt
GoalFields
- desiredOutcome
```

`desiredOutcome` is required when proposing a Goal. It should describe the outcome or direction in normal user language.

The AI may clarify or rewrite a user-provided Goal for review, but it must not silently replace the user's intended outcome.

A Goal draft does not contain an AI-calculated progress percentage.

#### Project Fields

```txt
ProjectFields
- completionMeaning?
```

`completionMeaning` may explain what would make the finite effort complete when this is useful and supported.

The AI must not create a Project only as a wrapper for one child item unless the user explicitly asked for a Project or the finite effort has independent meaning.

#### Task Fields

```txt
TaskFields
- plannedDate?
```

`plannedDate` is optional.

It may be included only when:

- the user explicitly provided a date
- the date follows deterministically from explicit context
- or the AI presents it as a visible assumption that the user may edit

A Task does not require a planned date merely because the draft contains a first-week view.

#### Routine Fields

```txt
RoutineFields
- recurrence
- preferredStartDate?
- preferredTime?
- durationMinutes?
```

`recurrence` is required for a Routine proposal.

The initial supported conceptual recurrence patterns are:

```txt
DAILY
WEEKDAYS
SPECIFIC_WEEKDAYS
WEEKLY
N_TIMES_PER_WEEK
MONTHLY_ON_DAY
```

Examples:

```txt
DAILY
Every day

WEEKDAYS
Monday through Friday

SPECIFIC_WEEKDAYS
Monday, Wednesday, Friday

WEEKLY
Every Friday

N_TIMES_PER_WEEK
Three times per week, without fixed occurrence dates yet

MONTHLY_ON_DAY
The first day of each month
```

Unsupported or ambiguous recurrence must not be silently converted to the nearest supported pattern.

Examples requiring warning, clarification, or partial output:

```txt
every so often
whenever I feel behind
three business days after every client reply
on alternating lunar Tuesdays, because apparently calendars were not already complicated enough
```

The exact recurrence encoding and occurrence-generation behavior belong to Discussions 015 and 019.

---

## 6. Assumptions

```txt
Assumption
- assumptionId
- text
- affectsDraftIds[]
- impact: LOW | MATERIAL
```

Assumptions must describe values or structures not explicitly confirmed by the user.

Examples:

```txt
I assumed “practice English regularly” means three times per week.
I treated “launch my portfolio” as a standalone Project rather than a Goal.
I left the application Tasks undated because no deadline was provided.
```

Rules:

- every material inferred ownership, date, recurrence, or entity classification must be visible
- assumptions may not conceal unsupported professional advice
- a draft with a material assumption remains reviewable, but the assumption must be prominent
- the user may edit or reject any assumption by editing the affected proposal

Low-impact wording cleanup does not require its own assumption entry.

---

## 7. Warnings

```txt
Warning
- code
- message
- affectsDraftIds[]
- severity: INFO | IMPORTANT | BLOCKING
```

Warnings identify contract or user-intent problems. They do not shame the user.

### INFO

The draft is valid, but the user should notice a limitation or omission.

Example:

```txt
No target date was provided, so Tasks are currently undated.
```

### IMPORTANT

The draft is usable, but approval should require explicit attention to the affected item.

Example:

```txt
The requested recurrence was interpreted as weekdays. Please verify it.
```

### BLOCKING

The affected proposal cannot be approved until corrected or removed.

Examples:

```txt
A Routine has no supported recurrence.
A parent reference is invalid.
A Task is attached to both a Goal and a Project.
A provided date is impossible or internally contradictory.
```

Safety crisis input does not produce a PlanningDraft with a blocking warning. It follows `SAFETY_FALLBACK` from Discussion 013 and produces no draft.

---

## 8. Unresolved Questions

```txt
UnresolvedQuestion
- questionId
- text
- affectsDraftIds[]
- blocking: true | false
```

This collection exists for partial drafts produced through `Draft now`, clarification exhaustion, or unsupported input portions.

It must not restart an unlimited interview inside the draft.

A blocking unresolved question prevents approval only for the affected proposal, not automatically for unrelated valid proposals.

Example:

```txt
What days should “three times per week” occur?
```

This is non-blocking if `N_TIMES_PER_WEEK` is supported without fixed weekdays.

Example:

```txt
Should this Routine belong to the existing Goal or the proposed Project?
```

This is blocking when ownership cannot be represented without choosing one.

---

## 9. Suggested First Week

```txt
SuggestedFirstWeek
- entries[]
```

```txt
FirstWeekEntry
- draftId
- date
- note?
```

The first-week schedule is optional.

It may be included when it helps the user understand immediate workload, especially for multi-item drafts.

It is not required for:

- a single standalone Task
- a single Routine with clear recurrence
- a draft with no meaningful timing information
- a long-term Goal whose initial work is not yet dated

Rules:

- it may reference proposed Tasks and Routines only
- it may not create separate canonical entities
- it must not directly create `RoutineOccurrence`
- it must remain editable and disappear if the referenced proposal is removed
- dates must remain consistent with Task planned dates and Routine recurrence

Discussion 015 determines how approved work enters Today and how occurrences are generated.

---

## 10. Numeric Limits

The limits are product guardrails for one planning response, not universal limits on what a user may eventually create.

### Proposed Default Limits per Draft

```txt
Goals: maximum 1
Projects: maximum 5
Tasks: maximum 15
Routines: maximum 5
Total proposals: maximum 20
First-week entries: maximum 21
Assumptions: maximum 10
Warnings: maximum 10
Unresolved questions: maximum 5
```

### Rationale

- one conversation should represent one coherent planning intention
- multiple Goals in one approval bundle create ambiguous scope and review complexity
- 15 Tasks permit a useful plan without generating a ceremonial backlog the user did not ask for
- 5 Routines reduce the risk of proposing an unrealistic lifestyle reconstruction in one click
- the user may request a smaller continuation draft later

If the request exceeds the limits, the AI should:

```txt
1. prioritize the most immediate coherent subset
2. state what was omitted
3. offer to prepare another draft after the current one is resolved
```

It must not silently truncate output.

---

## 11. Zero and Partial Output

### 11.1 Zero Tasks

Valid when:

- the user requested only a Routine
- the draft proposes only a Goal or Project definition pending later planning
- no Task can be proposed without inventing important details

### 11.2 Zero Routines

Valid when:

- the intention contains only one-off work
- recurrence would add no value
- the user explicitly asked for Tasks only

### 11.3 Zero Proposals

Valid only when:

- the planning input is unsupported but not a safety fallback
- contradictions prevent any coherent proposal
- the user requested `Draft now` before enough planning meaning existed

A zero-proposal response must include at least one warning or unresolved question explaining why no draft items were produced.

### 11.4 Partial Draft

A partial draft may contain valid proposals while omitting or blocking unresolved portions.

Example:

```txt
User asks for a portfolio Project and “some healthy routine” without defining the routine.

Valid partial result:
- portfolio Project and supported Tasks
- no invented health Routine
- unresolved question for the omitted Routine portion
```

---

## 12. Duplicate Handling

The AI should avoid duplicates within the draft through semantic comparison.

Examples of likely duplicates:

```txt
Update CV
Improve my resume

Practice English every weekday
Do English practice Monday to Friday
```

When two items express the same action:

- merge them when meaning is clearly equivalent
- preserve distinct items when their scope differs
- expose uncertainty rather than silently merging materially different work

Duplicate checking against existing canonical user data belongs partly to later product and data discussions. In this draft contract, existing-item conflicts may be represented as warnings, not silently resolved.

---

## 13. Validation Rules

A PlanningDraft is structurally valid only when:

- every proposal has a unique temporary `draftId`
- every `entityType` is supported
- every title is non-empty
- every parent reference points to a proposal in the same draft or to explicit existing context handled by the later API contract
- every parent mapping obeys Discussion 012
- no Task or Routine has both Goal and Project ownership
- every Routine has a supported recurrence or a blocking warning that prevents its approval
- dates are parseable and internally consistent
- counts remain within limits
- every first-week entry references an existing proposal
- warnings and assumptions reference valid draft items when references are present

### Semantic Validation

The system should also check:

- Project is not an artificial wrapper without independent meaning
- Goal does not merely restate a single Task
- Task is actionable and non-recurring
- Routine represents recurring behavior
- title and description do not contradict each other
- schedule does not contradict recurrence
- inferred content does not exceed user intent without visible assumptions

Semantic validation may use AI assistance, but deterministic contract rules take precedence.

---

## 14. Repair and Failure Behavior

Structured output validation occurs before the draft is shown as ready for approval.

Conceptual repair sequence:

```txt
1. validate deterministic structure
2. perform one bounded repair attempt for formatting or contract violations
3. revalidate
4. if still invalid, preserve the conversation and enter GENERATION_FAILED
```

The repair step may correct:

- malformed structured syntax
- missing optional containers
- invalid temporary references when the intended reference is unambiguous
- unsupported enum spelling when the intended supported value is exact and obvious

The repair step must not silently change:

- user intent
- entity ownership
- dates
- recurrence meaning
- Goal versus Project classification
- requested scope

Provider, retry count, schema library, parsing, and idempotency details belong to Discussions 018 and 020.

---

## 15. User Review Semantics

The user may edit before approval:

- titles
- descriptions
- entity classification
- parent ownership
- Task dates
- Routine recurrence and timing
- assumptions by editing the affected items
- inclusion or removal of any proposal
- first-week placement

The user may also request AI-assisted revision, but every revised result remains an unapproved draft.

### Approval Unit

The default interaction presents one draft for review, but approval occurs at item level within one confirmation action.

Before final confirmation, each proposal is in one of these review selections:

```txt
INCLUDED
EXCLUDED
BLOCKED
```

- `INCLUDED`: will be created after confirmation
- `EXCLUDED`: will not be created
- `BLOCKED`: cannot be included until corrected

The user may confirm all currently included and valid proposals together.

This avoids forcing one confirmation dialog per Task while still allowing selective approval.

### Parent Removal

If the user excludes a parent proposal:

- child proposals must not silently disappear
- the UI must ask or visibly offer valid alternatives

Possible outcomes:

```txt
exclude child too
make child standalone
move child to another valid included parent
```

The system must not silently reparent consequential structure.

### Partial Approval

Partial approval is allowed when the remaining included proposals are independently valid.

Examples:

- approve a Project and Tasks while excluding a proposed Routine
- approve standalone Tasks while excluding the proposed Goal
- exclude one invalid Task while approving unrelated valid items

Approval creates only the included, valid proposals.

---

## 16. Direct Answers to Original Questions

### 1. What exact fields exist in the planning draft?

The conceptual contract contains:

```txt
draftSummary
proposals[]
assumptions[]
warnings[]
unresolvedQuestions[]
suggestedFirstWeek?
```

Each proposal includes temporary identity, entity type, title, optional description, optional parent, source, confidence, and entity-specific fields.

### 2. Does AI create or only clarify the Goal?

It may propose a new Goal, clarify an explicitly stated Goal, or propose no Goal. Nothing is created before approval.

### 3. What are the maximum Task and Routine counts?

```txt
maximum 15 Tasks
maximum 5 Routines
```

### 4. Is a seven-day schedule required or optional?

Optional.

### 5. Which Tasks receive planned dates?

Only Tasks with explicit, deterministic, or visibly assumed dates.

### 6. Which recurrence patterns are supported?

```txt
DAILY
WEEKDAYS
SPECIFIC_WEEKDAYS
WEEKLY
N_TIMES_PER_WEEK
MONTHLY_ON_DAY
```

### 7. Are assumptions, rationale, and warnings included?

Assumptions and warnings are included. A separate verbose rationale field is excluded. `draftSummary` and visible assumptions provide sufficient explanation without encouraging AI to manufacture post-hoc reasoning.

### 8. Can the AI intentionally return zero Tasks or zero Routines?

Yes. Both are valid when unsupported by the intention.

### 9. How are invalid dates, unsupported recurrence, and duplicate items handled?

Through deterministic validation, warnings or blocking status, bounded repair for formatting only, and explicit duplicate merge or uncertainty handling.

### 10. How is structured output validated and repaired?

Deterministic validation first, one bounded non-semantic repair attempt, revalidation, then `GENERATION_FAILED` if still invalid.

### 11. What may the user edit before confirmation?

All consequential proposal fields, ownership, timing, inclusion, and first-week placement.

### 12. Is the whole draft confirmed together or item by item?

The user reviews items individually but confirms all included valid items in one final action. Partial approval is allowed.

---

## 17. Scenario Checks

### Scenario A — Standalone Task

```txt
User: Buy groceries tomorrow.
```

Expected:

```txt
one TASK
plannedDate = tomorrow
no Goal
no Project
no Routine
optional first-week view omitted
```

### Scenario B — Standalone Routine

```txt
User: Practice English for 30 minutes every weekday.
```

Expected:

```txt
one ROUTINE
recurrence = WEEKDAYS
durationMinutes = 30
zero Tasks
```

### Scenario C — Goal with Direct Work

```txt
User: I want to reach English B2. I should take a placement test and practice every weekday.
```

Expected:

```txt
one GOAL
one Goal-owned TASK
one Goal-owned ROUTINE
no forced Project
```

### Scenario D — Goal with Project

```txt
User: Help me find a frontend job. I need to build a portfolio and apply to companies.
```

Possible coherent result:

```txt
one GOAL
one PROJECT: Build portfolio
Project-owned Tasks
Goal-owned application Tasks or a second Project only if independently justified
```

The AI must expose any inferred separation through assumptions.

### Scenario E — Narrow Task Request

```txt
User: Email three companies tomorrow.
```

Expected:

```txt
one TASK or three Tasks depending on whether the companies are distinct known actions
no invented Goal
no invented job-search Project
```

### Scenario F — Oversized Request

```txt
User: Plan everything I need to do for the next six months to change careers.
```

Expected:

```txt
bounded first draft
maximum limits respected
visible omission warning
no silent 100-item backlog
```

### Scenario G — Unsupported Recurrence

```txt
User: Review my finances whenever the market feels unstable.
```

Expected:

```txt
no silently invented recurrence
partial draft or blocking warning
no financial advice
```

### Scenario H — Crisis Input

Expected:

```txt
no PlanningDraft
route to SAFETY_FALLBACK from Discussion 013
```

---

## 18. Included, Excluded, and Deferred

### Included Now

- conceptual PlanningDraft contract
- supported proposal types
- temporary parent relationships
- assumptions, warnings, and unresolved questions
- optional first-week view
- numeric limits
- zero and partial output behavior
- deterministic validation boundaries
- bounded formatting repair
- selective item review and partial approval

### Explicitly Excluded

- persisted `Plan` entity
- direct RoutineOccurrence creation
- hidden AI actions
- unlimited generated backlogs
- mandatory seven-day scheduling
- verbose chain-of-thought or rationale field
- automatic approval
- one confirmation dialog per proposed item

### Deferred

- exact JSON Schema and DTO syntax
- IDs for existing canonical context
- persistence and transaction behavior
- occurrence generation
- duplicate detection against existing stored data
- runtime repair implementation
- analytics events
- provider and retry architecture

---

## 19. Mind Map Impact — Record Only, Do Not Apply Yet

After Discussion 021 consolidation, the Mind Map should later add or revise:

### AI Responsibilities

- produce bounded structured PlanningDrafts
- map output only to Goal, Project, Task, and Routine
- expose assumptions and warnings
- generate partial drafts without fabricating missing content
- respect numeric limits
- validate ownership and recurrence before review
- avoid silent semantic repair

### User Flow

Add:

```txt
Draft generated
→ item-level review
→ edit / include / exclude / resolve blocked items
→ one final confirmation
→ create included valid entities only
```

### AI Guardrails

Add:

- no hidden creation before approval
- no mandatory Goal or Project wrapper
- no unsupported recurrence conversion
- no silent truncation of oversized drafts
- no silent reparenting when a parent is excluded
- no PlanningDraft for safety-fallback input
- no verbose hidden rationale requirement

### Open Questions to Remove or Narrow

- whether first-week schedule is mandatory
- whether zero-Task or zero-Routine output is valid
- whether approval is all-or-nothing
- whether AI may propose a Goal
- whether assumptions and warnings are part of the draft
- what default output limits apply

No Mind Map change is applied yet.

---

## 20. Affected Formal Documents — Record Only, Do Not Update Yet

After Discussion 021 consolidation, accepted decisions from this discussion should update or create:

- PlanningDraft product contract
- AI planning responsibility specification
- draft review and approval UX specification
- AI guardrail specification
- Today intake specification in coordination with Discussion 015
- data model and transaction specification in Discussion 019
- runtime structured-output and API specification in Discussion 020
- validation plan in Discussion 021
- implementation plan in Discussion 022

Potential ADR impact:

- an ADR may be required for ephemeral PlanningDraft plus selective approval semantics
- an ADR may be required if structured-output repair strategy materially constrains provider architecture

No formal document is updated from this discussion alone before consolidation after Discussion 021.

---

## 21. Structured Review Request for Claude

Please review only the structural coherence of this proposed PlanningDraft contract.

For every issue, provide:

```txt
1. affected decision
2. concrete failure scenario
3. severity: blocking, important, or minor
4. smallest coherent correction
```

Focus on:

- contradictions with accepted Discussions 012 and 013
- proposal relationships that cannot represent common cases
- missing fields that are necessary for review but still within this discussion's scope
- fields that add complexity without a defined product meaning
- numeric limits that break ordinary planning scenarios
- invalid or unsafe partial-approval behavior
- ambiguity between assumptions, warnings, and unresolved questions
- contract rules that cannot be validated consistently
- silent semantic changes during repair
- cases where excluding a parent corrupts child meaning

Do not expand the review into:

- database tables or constraints
- API implementation
- provider selection
- prompt wording
- analytics
- occurrence generation
- Today execution
- Reconcile
- implementation sequencing

This discussion must remain open until Claude reviews this exact version and accepted corrections, final resolution, Mind Map impact, and affected formal documents are recorded.