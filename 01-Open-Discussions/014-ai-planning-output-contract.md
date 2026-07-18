# Discussion 014 — AI Planning Output Contract

## Status

Open for GPT × Claude review.

This version contains GPT's proposed decisions, including Mahdi's explicit one-Goal and one-week planning constraints. It is not yet accepted.

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
How is the hierarchy shown for review?
What may the user edit, include, exclude, and approve?
What happens to children when a proposed parent is rejected?
What is the maximum actionable planning horizon?
```

The draft is ephemeral. It is not a canonical product entity and creates nothing by itself.

Core invariant inherited from Discussion 013:

> Conversation produces a proposal. Approval creates product entities.

---

## 2. Explicitly Out of Scope

This discussion does not define:

- conversation entry or clarification limits — Discussion 013
- visual styling, exact component layout, animation, or responsive implementation
- persistence tables, constraints, transactions, or events — Discussion 019
- provider, runtime, retry, transport, or schema-enforcement implementation — Discussion 020
- Today execution, occurrence generation, or Carry — Discussion 015
- detailed weekly-review trigger and adaptation flow — Discussions 015–017
- detailed safety detection or provider-failure policy — Discussion 018
- implementation sequencing — Discussion 022

This discussion does define the UX semantics required to review the contract coherently. Later UX specifications may choose cards, trees, accordions, or other visual components, but may not change the accepted review behavior silently.

---

## 3. Accepted Dependencies

Discussion 012 established the canonical concepts:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

Planning AI may propose:

```txt
Goal
Project
Task
Routine
```

It does not directly propose `RoutineOccurrence`.

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
- material assumptions must be visible
- no canonical entities exist before approval
- safety fallback produces no PlanningDraft

---

## 4. Proposed Contract Principles

### 4.1 Draft, Not Hidden Execution

Planning AI returns a proposal that the product can render, edit, validate, and approve.

It must not:

- create canonical entities before approval
- return hidden actions that bypass review
- invent product concepts outside Discussion 012
- silently change a materially different user intention
- claim proposed dates, recurrence, or ownership were already applied

### 4.2 One Coherent Intention and Maximum One Goal

One PlanningDraft represents one coherent planning intention.

Planning AI may propose:

```txt
zero Goals
or exactly one Goal
```

It must never propose more than one Goal in a single draft.

When the input contains multiple independent desired outcomes, the AI must:

```txt
1. ask the user to choose one Goal
or
2. produce a draft for the most immediate coherent intention with a visible omission
or
3. propose separate future planning flows
```

It must not bundle several Goals into one approval package merely because they appeared in one message.

This limit applies to proposed new Goals. A contextual planning flow may reference one existing Goal without proposing another unrelated Goal.

### 4.3 Minimal Complete Structure

The draft contains only structures justified by the intention.

Valid forms include:

```txt
one standalone Task
one standalone Routine
one standalone Project with child work
one Goal with direct Tasks and Routines
one Goal with Projects and child work
multiple standalone Tasks under one coherent intention
```

The AI must not create Goal or Project wrappers merely to make the output look more organized.

### 4.4 Long-Term Direction, One-Week Action Horizon

A Goal or Project may represent work lasting months or years.

However, one PlanningDraft may schedule actionable work for a maximum horizon of seven calendar days.

```txt
Long-term Goal or Project structure
+ maximum seven-day actionable plan
```

Even when the user asks for a one-year plan, the AI must not create a fully scheduled one-year execution backlog.

It should instead provide:

- the long-term Goal or Project structure justified by the request
- only the Tasks and Routine placements that are actionable or reviewable for the first seven days
- an explicit note that later weeks require renewed confirmation

The one-week limit is not a limit on Goal duration, Project duration, or Routine lifetime. It limits only the currently approved detailed execution horizon.

### 4.5 Weekly Continuation Requires User Confirmation

At the beginning of the next planning week, the system must not silently roll the same detailed plan forward as if it remains suitable.

The product must obtain a lightweight user decision:

```txt
Continue this plan as-is
Adjust the plan
Pause or stop relevant work
Review with AI
```

The exact trigger timing, Today integration, evidence used for adaptation, and Reconcile behavior belong to Discussions 015–017.

The invariant established here is:

> No detailed weekly execution plan becomes the next week's accepted plan without user confirmation.

A Routine may remain a canonical recurring definition after approval, but its next-week planning presentation and any AI-proposed adaptation still require the weekly review behavior defined later.

---

## 5. Proposed PlanningDraft Shape

```txt
PlanningDraft
- draftSummary
- proposals[]
- assumptions[]
- warnings[]
- unresolvedQuestions[]
- firstWeekPlan?
- continuationPolicy
```

### 5.1 `draftSummary`

A short user-facing summary of what the AI understood and proposed.

Rules:

- one to three short sentences
- no claim that the Goal is guaranteed or objectively achievable
- no personality or motivational diagnosis
- no hidden decision absent from structured proposals
- when the intention is long-term, state that detailed scheduling covers only the first week

### 5.2 `proposals[]`

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

#### Temporary Identity

`draftId` connects proposals inside the ephemeral draft. It is not a canonical product ID.

#### Parent Relationships

Allowed mappings:

```txt
PROJECT → GOAL
TASK → GOAL | PROJECT
ROUTINE → GOAL | PROJECT
```

A Task or Routine with no parent is standalone.

Forbidden mappings:

```txt
GOAL → any parent
PROJECT → PROJECT
TASK → TASK | ROUTINE
ROUTINE → TASK | ROUTINE
```

#### Source

```txt
EXPLICIT
The proposal directly reflects user input.

INFERRED
The AI introduced structure beyond literal wording.
```

Inferred structure must remain visible during review.

#### Confidence

Confidence describes interpretation confidence, not success probability.

```txt
HIGH
Directly follows explicit input.

MEDIUM
Reasonable interpretation with limited ambiguity.

LOW
Depends on a material assumption or likely correction.
```

### 5.3 Goal Fields

```txt
GoalFields
- desiredOutcome
```

Rules:

- required when proposing a Goal
- maximum one proposed Goal per PlanningDraft
- may clarify the user's outcome but must not replace it silently
- contains no AI-calculated achievement percentage

### 5.4 Project Fields

```txt
ProjectFields
- completionMeaning?
```

A Project must represent a finite or independently manageable effort. It must not exist only as an artificial wrapper.

### 5.5 Task Fields

```txt
TaskFields
- plannedDate?
```

A planned date may appear only when:

- explicitly supplied
- deterministically derived from confirmed context
- or shown as a visible editable assumption

Any proposed `plannedDate` must fall within the current seven-day actionable horizon. Longer-term work may remain undated or represented at Project level.

### 5.6 Routine Fields

```txt
RoutineFields
- recurrence
- preferredStartDate?
- preferredTime?
- durationMinutes?
```

Supported conceptual recurrence patterns:

```txt
DAILY
WEEKDAYS
SPECIFIC_WEEKDAYS
WEEKLY
N_TIMES_PER_WEEK
MONTHLY_ON_DAY
```

A Routine definition may describe recurrence beyond one week, but the detailed first-week plan may show only occurrences or placements within seven days.

Unsupported recurrence must not be silently converted to a nearby supported pattern.

### 5.7 `continuationPolicy`

```txt
ContinuationPolicy
- reviewRequiredAtNextWeek: true
- allowedChoices:
  - CONTINUE_AS_IS
  - ADJUST
  - PAUSE_OR_STOP
  - REVIEW_WITH_AI
```

For AI-generated multi-day or long-term plans, `reviewRequiredAtNextWeek` is always `true`.

This field records the product invariant. Exact persistence and triggering belong to later discussions.

---

## 6. Assumptions

```txt
Assumption
- assumptionId
- text
- affectsDraftIds[]
- impact: LOW | MATERIAL
```

Every material inferred ownership, date, recurrence, entity classification, or reparenting choice must be visible.

The user resolves an assumption by editing, accepting, or excluding the affected proposal.

---

## 7. Warnings

```txt
Warning
- code
- message
- affectsDraftIds[]
- severity: INFO | IMPORTANT | BLOCKING
```

`BLOCKING` prevents approval of the affected proposal until corrected or excluded.

Examples:

- invalid parent reference
- Task or Routine assigned to both Goal and Project
- unsupported recurrence
- impossible date
- proposed Task date beyond the current seven-day planning horizon
- more than one proposed Goal

Safety-crisis input produces no PlanningDraft and routes to `SAFETY_FALLBACK` from Discussion 013.

---

## 8. Unresolved Questions

```txt
UnresolvedQuestion
- questionId
- text
- affectsDraftIds[]
- blocking: true | false
```

Unresolved questions are used for partial drafts. They must not restart an unlimited interview.

A blocking question prevents approval only for affected proposals, unless the unresolved issue corrupts the whole hierarchy.

---

## 9. First-Week Plan

```txt
FirstWeekPlan
- startDate
- endDate
- entries[]
```

```txt
FirstWeekEntry
- draftId
- date
- note?
```

### 9.1 Horizon Rule

The detailed plan covers no more than seven consecutive calendar days.

```txt
endDate - startDate <= 6 days
```

The AI must not create detailed week-two or later placements inside the same initial approval draft.

### 9.2 When It Is Required

A first-week plan is required when the AI proposes a multi-step or long-term plan with actionable Tasks or Routines.

It may be omitted for:

- one standalone Task with an explicit date
- one standalone Routine whose recurrence is already fully clear
- a Goal or Project definition with no actionable work yet
- a partial draft that cannot safely schedule work

### 9.3 Rules

- references Tasks and Routines only
- creates no separate canonical entity
- creates no `RoutineOccurrence` directly
- remains editable before approval
- disappears or updates when referenced proposals are excluded or edited
- must remain consistent with Task dates and Routine recurrence
- cannot contain entries outside the seven-day horizon
- must state that next-week continuation requires user confirmation

---

## 10. Numeric Limits

Limits per PlanningDraft:

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
Detailed planning horizon: maximum 7 calendar days
```

These are limits on one AI draft, not universal limits on canonical user data.

When input exceeds limits, the AI must prioritize a coherent immediate subset, state what was omitted, and offer another planning flow later. It must not silently truncate.

---

## 11. Zero and Partial Output

Zero Tasks or zero Routines are valid when the intention does not justify them.

Zero proposals are valid only when no coherent supported proposal can be produced. The result must explain why through warnings or unresolved questions.

A partial draft may contain valid proposals while omitting or blocking unresolved portions.

The AI must not generate filler Tasks merely to occupy all seven days.

---

## 12. Duplicate Handling

The AI should avoid semantic duplicates within the draft.

It may merge only when meaning is clearly equivalent. Uncertain or materially different items must remain separate or be surfaced for review.

Conflicts with existing canonical data may be represented as warnings; silent replacement or merging is forbidden.

---

## 13. Validation Rules

A PlanningDraft is structurally valid only when:

- every proposal has a unique `draftId`
- every entity type is supported
- no more than one Goal is proposed
- every title is non-empty
- parent references are valid
- parent mappings obey Discussion 012
- no Task or Routine has both Goal and Project ownership
- every Routine has supported recurrence or is blocked
- dates are valid and consistent
- actionable dates and first-week entries stay inside seven days
- counts remain within limits
- warnings, assumptions, and questions reference valid items
- first-week entries reference included Task or Routine proposals
- continuation policy requires next-week review where applicable

Semantic validation should also check:

- Project is not an artificial wrapper
- Goal is not merely a renamed Task
- Task is actionable and non-recurring
- Routine represents recurring behavior
- hierarchy and descriptions do not contradict each other
- the first week is not unrealistically overloaded
- inferred structure remains visible

Deterministic contract rules take precedence over AI semantic judgment.

---

## 14. Repair and Failure Behavior

Conceptual repair sequence:

```txt
1. deterministic validation
2. one bounded structural repair attempt
3. revalidation
4. GENERATION_FAILED if still invalid
```

Repair may fix formatting, missing optional containers, obvious enum spelling, or unambiguous temporary references.

Repair must not silently change:

- user intent
- Goal versus Project classification
- ownership
- dates
- recurrence meaning
- requested scope
- parent-removal outcome
- the seven-day horizon

---

## 15. Draft Review and Relationship UX Semantics

### 15.1 Hierarchy Must Be Visible

The review experience must present ownership clearly enough that a normal user can understand:

```txt
Goal
├── Project
│   ├── Task
│   └── Routine
├── direct Task
└── direct Routine
```

The exact visual component is deferred, but the hierarchy must not be flattened into an ambiguous list.

Each item must visibly communicate:

- entity type
- parent or standalone status
- included, excluded, or blocked state
- material assumptions or warnings attached to it
- first-week placements where relevant

### 15.2 Edit Before Approval

The user edits the draft before canonical creation.

Editable fields include:

- title and description
- entity classification
- ownership
- Task date
- Routine recurrence and timing
- first-week placement
- include or exclude selection

AI-assisted revision is allowed, but every revision remains unapproved until final confirmation.

The product must not require the user to approve unwanted entities first and repair them after creation.

### 15.3 Review Selection States

Each proposal has one review state:

```txt
INCLUDED
EXCLUDED
BLOCKED
```

- `INCLUDED`: valid and selected for creation
- `EXCLUDED`: will not be created
- `BLOCKED`: cannot be included until corrected

### 15.4 Approval Unit

The user reviews items individually but performs one final confirmation for all currently included and valid proposals.

```txt
item-level review
→ edit / include / exclude
→ resolve structural conflicts
→ one final explicit approval
```

This avoids both blind all-or-nothing approval and one confirmation dialog per Task.

Partial approval is allowed when the remaining included hierarchy is valid.

### 15.5 Parent Exclusion Must Never Be Silent

When a Goal or Project is excluded, its children must not silently disappear or silently receive a new owner.

The product must enter an explicit dependency-resolution interaction.

Valid choices may include:

```txt
1. exclude the parent and all selected descendants
2. keep eligible children as standalone items
3. move eligible children to another valid included parent
4. cancel parent exclusion
```

Only semantically valid options should be shown.

Examples:

- a Project may become standalone after its Goal is excluded
- a Task or Routine may become standalone when allowed
- a Project-owned Task may move to the parent Goal only with explicit user selection
- a Project-owned Routine may be moved only after making the changed lifecycle meaning visible

The system must revalidate all affected items after any reparenting.

### 15.6 Child Exclusion

Excluding a child does not exclude its parent.

If excluding all children leaves an inferred Project with no independent meaning, the system should warn that the Project may now be an artificial empty wrapper. It must not remove it silently.

### 15.7 Questions and Warnings in Review

Warnings, assumptions, and unresolved questions should appear beside or within the affected item, not only in one detached global list.

Global summaries may exist, but they do not replace item-level context.

### 15.8 Exact Visual Design Deferred

Deferred UX details include:

- tree versus nested cards
- accordion behavior
- mobile navigation
- drag-and-drop
- modal versus inline editing
- icons, spacing, colors, animation, and wording polish

A later UX specification may decide these details but must preserve the semantics above.

---

## 16. Direct Answers to Original Questions

### What exact fields exist?

```txt
draftSummary
proposals[]
assumptions[]
warnings[]
unresolvedQuestions[]
firstWeekPlan?
continuationPolicy
```

### Does AI create or only clarify a Goal?

It may propose zero or one Goal. It may never propose more than one Goal per draft. Nothing is created before approval.

### What are the maximum Task and Routine counts?

```txt
15 Tasks
5 Routines
```

### Is a seven-day plan required?

A detailed first-week plan is required for multi-step or long-term actionable drafts. The maximum detailed horizon is seven calendar days.

### Can a Goal last longer than a week?

Yes. Goal, Project, and Routine lifetimes may be long-term. Only the detailed approved execution horizon is capped at one week.

### Which Tasks receive planned dates?

Only Tasks with explicit, deterministic, or visible assumed dates, and only inside the current seven-day horizon.

### Can AI return zero Tasks or zero Routines?

Yes, when they are not justified by the intention.

### What may the user edit?

All consequential proposal fields, ownership, timing, inclusion, and first-week placement before final approval.

### Is approval whole-draft or item-by-item?

Items are reviewed individually, but all included valid items are confirmed in one final action. Partial approval is allowed.

### What happens when a parent is rejected?

The system asks the user how to handle descendants. It never silently deletes or reparents them.

### What happens next week?

The system asks whether the user wants to continue, adjust, pause/stop, or review with AI. Detailed weekly-review mechanics are finalized later.

---

## 17. Scenario Checks

### Scenario A — Standalone Task

```txt
User: Buy groceries tomorrow.
```

Expected:

```txt
one standalone Task
no Goal
no Project
firstWeekPlan may be omitted
```

### Scenario B — One-Year Learning Goal

```txt
User: Make me a one-year plan to reach English B2.
```

Expected:

```txt
maximum one Goal
long-term outcome preserved
Projects may describe major finite efforts
only first seven days receive detailed Task or Routine placement
no 365-day generated backlog
continuationPolicy requires review next week
```

### Scenario C — Multiple Goals in One Message

```txt
User: Help me learn English, find a new job, and get fit.
```

Expected:

```txt
no three-Goal draft
ask the user to choose one intention
or produce one explicitly prioritized Goal draft with visible omissions
```

### Scenario D — Excluding a Project

```txt
Goal: Find a frontend job
Project: Build portfolio
- Task: Buy domain
- Task: Write case study
- Routine: Work on portfolio weekdays
```

If the user excludes `Build portfolio`, the product must ask what to do with its children. No descendant is silently removed or moved.

### Scenario E — Excluding a Goal

If the Goal is excluded, child Projects, Tasks, and Routines may be excluded or explicitly converted to valid standalone structures. No automatic conversion occurs.

### Scenario F — Oversized Request

```txt
Plan every action I need for the next year.
```

Expected:

```txt
bounded structure
maximum one Goal
maximum seven-day detailed plan
visible omitted-future-work notice
no ceremonial backlog
```

### Scenario G — Crisis Input

```txt
no PlanningDraft
route to SAFETY_FALLBACK
```

---

## 18. Included, Excluded, and Deferred

### Included Now

- conceptual PlanningDraft contract
- maximum one proposed Goal
- seven-day maximum detailed planning horizon
- mandatory next-week confirmation invariant
- hierarchy visibility requirements
- edit-before-approval behavior
- item selection and one final confirmation
- partial approval
- explicit parent-exclusion resolution
- assumptions, warnings, and unresolved questions
- numeric limits
- zero and partial output
- validation and bounded structural repair

### Explicitly Excluded

- persisted `Plan` entity
- direct RoutineOccurrence creation
- more than one proposed Goal per draft
- detailed scheduling beyond seven days
- silent next-week continuation
- silent parent cascade or reparenting
- hidden AI actions
- unlimited generated backlogs
- verbose hidden rationale or chain-of-thought
- automatic approval
- one confirmation dialog per proposed item

### Deferred

- exact visual components and wireframes
- exact weekly review trigger and notification timing
- evidence shown during weekly review
- adaptation and Reconcile behavior
- exact JSON Schema and DTO syntax
- persistence, transactions, and events
- duplicate detection against stored data
- provider and runtime repair implementation
- analytics

---

## 19. Mind Map Impact — Record Only, Do Not Apply Yet

After Discussion 021 consolidation, add or revise:

### AI Responsibilities

- produce bounded structured PlanningDrafts
- propose at most one Goal per draft
- preserve long-term direction while planning no more than seven detailed days
- expose assumptions, warnings, and unresolved questions
- generate partial drafts without fabricating content
- validate hierarchy and recurrence
- avoid silent reparenting and semantic repair

### User Flow

```txt
Draft generated
→ hierarchy review
→ edit / include / exclude
→ resolve parent-child conflicts
→ review first-week plan
→ one final confirmation
→ create included valid entities
→ next-week confirmation before detailed continuation
```

### AI Guardrails

- no hidden creation before approval
- no multiple-Goal draft
- no detailed schedule beyond seven days
- no silent weekly rollover
- no unsupported recurrence conversion
- no silent truncation
- no silent deletion or reparenting of descendants
- no PlanningDraft for safety fallback

### Open Questions to Remove or Narrow

- whether more than one Goal may be proposed
- whether first-week scheduling is optional for long-term plans
- whether approval is all-or-nothing
- whether editing happens before or after creation
- what happens when a parent is excluded
- whether future weeks are generated in advance

No Mind Map change is applied yet.

---

## 20. Affected Formal Documents — Record Only, Do Not Update Yet

After Discussion 021 consolidation, accepted decisions should update or create:

- PlanningDraft product contract
- AI planning responsibility specification
- draft review and approval UX specification
- hierarchy and dependency-resolution UX specification
- weekly planning and continuation specification
- AI guardrail specification
- Today intake specification with Discussion 015
- weekly review and adaptation specification with Discussions 015–017
- data model and transaction specification in Discussion 019
- runtime structured-output and API specification in Discussion 020
- validation plan in Discussion 021
- implementation plan in Discussion 022

Potential ADRs:

- ephemeral PlanningDraft plus selective approval
- one-Goal-per-draft boundary
- seven-day detailed planning horizon and weekly user confirmation

No formal document is updated before consolidation after Discussion 021.

---

## 21. Structured Review Request for Claude

Please review only the structural coherence of this proposed PlanningDraft and review contract.

For every issue, provide:

```txt
1. affected decision
2. concrete failure scenario
3. severity: blocking, important, or minor
4. smallest coherent correction
```

Focus on:

- contradictions with accepted Discussions 012 and 013
- whether maximum one Goal per draft breaks common coherent scenarios
- whether long-term Goals and Projects remain representable with a seven-day detailed horizon
- ambiguity between canonical Routine recurrence and weekly confirmation
- whether the next-week confirmation invariant is placed in the correct discussion without overdefining 015–017
- hierarchy fields necessary for understandable review
- parent-exclusion cases that may corrupt descendant meaning
- invalid partial-approval outcomes
- whether item-level review plus one final confirmation is coherent
- ambiguity between assumptions, warnings, and unresolved questions
- validation rules that cannot be enforced consistently
- any UX semantic rule that is actually presentation-only and should be deferred

Do not expand the review into:

- visual styling or wireframes
- database tables
- API implementation
- provider selection
- prompt wording
- analytics
- occurrence generation details
- full Today execution mechanics
- detailed Reconcile behavior
- implementation sequencing

This discussion must remain open until Claude reviews this exact version and accepted corrections, final resolution, Mind Map impact, and affected formal documents are recorded.
