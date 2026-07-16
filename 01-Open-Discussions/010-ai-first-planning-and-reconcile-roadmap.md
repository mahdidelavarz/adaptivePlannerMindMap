# Discussion 010 — AI-First Planning Before AI-Assisted Reconcile

## Status

Open for GPT × Claude review.

This discussion intentionally reopens part of the accepted Phase 1 product scope. It does **not** silently override:

- [[02-Decisions/ADR-003-phase-0-closure]]
- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/api-contracts-phase-1]]
- [[05-Implementation/phase-1-plan]]

If the direction below is accepted, the affected ADRs, specs, validation plan, and implementation plan must be updated explicitly.

---

## 1. Why This Discussion Was Opened

The accepted Phase 1 was optimized around one hypothesis:

```txt
Create Tasks
→ execute or miss them
→ return later
→ resolve unfinished work through Reconcile
```

During review of the long-absence case, this scope exposed a more fundamental problem.

If the user returns after five days with 20 unresolved items, or after one month with 100 items, item-by-item `Done / Carry / Drop` becomes a second backlog. It reproduces the decision fatigue the product is supposed to reduce.

That led to work on:

- bulk recovery
- recurring tasks / routines
- goal-level recovery
- project or path abstractions
- AI-assisted grouping and prioritization

The deeper strategic issue then became clear:

> Reconcile is a late-stage recovery mechanism. The user first has to create a useful plan, understand what to do, and receive enough daily value to generate meaningful execution data.

The current Phase 1 asks the user to fill a mostly empty planner manually, then differentiates itself only after the plan has already failed. That risks testing a weak Todo product rather than testing the real Adaptive Planner thesis.

---

## 2. Research Summary: The Reconcile Problem Is Real, but the Basic Mechanism Is Not New

### 2.1 What is supported

The research reviewed so far supports the following claims:

1. Unfinished goals and unresolved work can remain cognitively active.
2. Making a concrete plan for unfinished work can reduce intrusive thoughts.
3. Users and products repeatedly need mechanisms for rollover, rescheduling, archiving, reset, and recovery.
4. Large overdue lists can create decision fatigue, avoidance, and loss of trust in the plan.
5. Recurring work must not accumulate exactly like standalone unfinished Tasks.

A relevant experimental paper is:

- Masicampo, E. J., & Baumeister, R. F. (2011), *Consider it done! Plan making can eliminate the cognitive effects of unfulfilled goals*. The studies found that forming a specific plan for an unfinished goal reduced intrusive cognitive effects associated with the unfinished goal.
  - PubMed: https://pubmed.ncbi.nlm.nih.gov/21688924/
  - DOI: https://doi.org/10.1037/a0024192

The Fresh Start literature also supports the broader idea that meaningful temporal boundaries can help users restart goal pursuit instead of remaining trapped in prior failure:

- Dai, H., Milkman, K. L., & Riis, J. (2014), *The Fresh Start Effect: Temporal Landmarks Motivate Aspirational Behavior*.
  - Open-access copy: https://pmc.ncbi.nlm.nih.gov/articles/PMC4839284/

These sources support the pain and the usefulness of making a new plan. They do **not** prove that our exact `Done / Carry / Drop` flow improves retention.

### 2.2 Direct product precedents

The core Reconcile interaction already exists in nearby forms.

#### Structured — Replan

Structured explicitly allows users to revisit unfinished tasks from the past and decide whether to:

- check them off
- delete them
- reschedule them
- move them to Inbox

Source:

- https://help.structured.app/en/articles/1990594

Structured also now exposes AI-assisted schedule changes. Its help documentation describes asking Structured AI to filter unfinished tasks and apply requested changes, such as moving most unfinished Tasks to tomorrow while moving one specific Task to Inbox.

This matters because:

```txt
Item-level Reconcile already has direct market precedent.
AI-assisted backlog manipulation also has direct precedent.
```

#### Sunsama — Rollover and Archive

Sunsama automatically rolls incomplete Tasks forward and automatically archives Tasks that have rolled over repeatedly. The archive exists specifically to declutter work that the user is not doing.

Source:

- https://help.sunsama.com/docs/getting-started/basics/task-rollover-and-recurring-tasks-the-basics/

Sunsama also treats recurring Tasks differently. When an unfinished recurring instance collides with the next instance, the older unchanged instance is removed so duplicate routine debt does not accumulate.

Source:

- https://help.sunsama.com/docs/usage-guides/tasks/recurring-tasks/

This supports an important domain decision reached during this review:

```txt
A missed Routine occurrence is not equivalent to an overdue standalone Task.
Past Routine occurrences should not be Carried into today as accumulated debt.
```

### 2.3 What is not proven

We did not find public controlled evidence that a dedicated return-time flow using `Done / Carry / Drop`:

- reduces guilt better than Fresh Start
- increases return rate
- improves long-term retention
- performs better than rollover, archive, or automatic rescheduling

Competitors may have internal A/B data, but public product documentation does not provide it.

We also reviewed AI-generated reports that presented dozens of precise user quotes and physiological claims. Many did not include traceable direct URLs, and several exact quotes could not be independently located. Therefore:

```txt
Unlinked social quotes are context, not accepted evidence.
Claims such as red badges directly raising cortisol are rejected unless a direct study supports them.
```

### 2.4 Current research verdict

```txt
Pain: real and supported
Basic Replan interaction: already exists
Routine-aware handling: already exists in partial form
Retention impact: unproven
Differentiation of our current Phase 1: weak unless the system becomes more adaptive
```

---

## 3. Why AI Eventually Belongs Inside Reconcile

A large backlog is not mainly a rendering problem. It is a classification and decision-compression problem.

A useful intelligent Reconcile system should be able to:

- separate explicit Routine occurrences from standalone Tasks
- group semantically similar Tasks
- identify likely duplicates
- separate protected items with deadlines from low-risk stale work
- summarize multiple items under a common intent
- detect repeated Carry patterns
- detect when the problem is no longer one Task but the current plan or goal
- propose a small number of meaningful decisions instead of presenting 50 separate cards

The intended model is:

```txt
Rules establish facts
→ structured data creates safe groups
→ AI interprets ambiguous meaning
→ user confirms consequential decisions
```

Examples:

```txt
Rule-based certainty:
Past occurrence of an explicit Routine → Missed

Structured grouping:
Same routineId / goalId / normalized title / date range

AI inference:
These five differently-worded Tasks probably express the same unfinished intent

User-owned decision:
Keep one, keep all, move to backlog, or let go
```

### 3.1 AI responsibilities in future Reconcile

AI may:

- classify
- group
- summarize
- prioritize for review
- explain why items were grouped
- recommend bulk actions
- suggest reviewing the broader plan

AI must not silently:

- Drop important Tasks
- abandon a Goal
- delete history
- rewrite multiple future dates
- make medical, financial, legal, or other high-risk decisions

The governing rule is:

```txt
AI reduces the decision space.
The user owns the decision.
```

### 3.2 Why this is not the first AI feature to build

AI-assisted Reconcile needs meaningful historical data:

- actual Tasks
- actual Routine occurrences
- Carry patterns
- Missed patterns
- accepted and rejected plans
- evidence that a plan no longer fits reality

Building this first means creating an intelligent cleanup system before proving that the product can create a plan worth executing.

Therefore AI-assisted Reconcile remains strategically important, but it should follow initial AI-assisted plan creation.

---

## 4. The Earlier Problem: The Empty Planner

The current Day-0 experience still asks the user to do most of the difficult planning work:

```txt
Know what the Goal means
→ break it into executable work
→ decide which actions repeat
→ estimate realistic frequency
→ choose what belongs in the first week
→ enter all of it through forms
```

This creates an absurd product contract:

> The user comes because planning is difficult; the application gives them forms and asks them to finish the planning alone.

The first value moment should not be a successful form submission. It should be:

> “I explained what I want, and now my next steps are clearer and usable.”

This problem occurs at activation, before Reconcile, and directly determines whether useful Tasks and Routines exist at all.

---

## 5. New Proposed Phase 1 Hypothesis

### Primary hypothesis

> AI-assisted setup can reduce the effort of turning a user intention into a small, credible set of Tasks and Routines that the user is willing to execute.

### Secondary hypothesis

> Once enough real execution history exists, AI-assisted Reconcile can later reduce recovery decision fatigue by grouping and prioritizing unresolved work.

This changes the ordering from:

```txt
Manual plan creation
→ basic execution
→ Reconcile as core differentiation
```

To:

```txt
AI-assisted plan creation
→ daily execution
→ collect real behavior
→ AI-assisted adaptation and Reconcile
```

Reconcile is not rejected. It is moved from the first value engine to the later adaptation engine.

---

## 6. Proposed AI Scope for the First Implementation

This must not become an unrestricted AI life coach.

The first implementation should solve a bounded problem:

> Make it easier to create the first useful Tasks and Routines.

### 6.1 User input

The assistant may ask for a small set of clarifying inputs:

- What do you want to achieve or organize?
- What is your current situation?
- What time can you realistically spend?
- Is there a target date?
- What constraints matter?
- What have you already tried or already completed?

The interface should not become a long questionnaire. AI should ask only questions whose answers materially change the proposed plan.

### 6.2 AI output

For the first iteration, the AI should produce a deliberately small draft:

- a clarified summary of the user intention
- assumptions that need confirmation
- zero to three initial one-time Tasks
- zero to two Routines
- Routine frequencies supported by the product
- a short plan for the next seven days

Supported Routine scheduling remains intentionally limited:

```txt
Daily
Selected days of week
Selected day of month
```

The AI should not generate a detailed multi-month schedule or hundreds of future Task rows.

### 6.3 User control

Every proposal must be:

- reviewable
- editable
- removable
- individually selectable
- confirmed before persistence

The required interaction rule is:

```txt
AI proposes
→ user edits and approves
→ system creates Tasks and Routines
```

### 6.4 What is explicitly outside the first AI scope

- autonomous background plan changes
- personality diagnosis
- unrestricted life coaching
- detailed long-term project management
- medical, legal, or financial planning without dedicated guardrails
- automatic Goal abandonment
- automatic Drop decisions
- fully autonomous calendar scheduling
- AI-generated 30/60/180-day Task lists
- complex dependency graphs

### 6.5 What remains without AI in the first implementation

A minimal non-intelligent recovery rule should still prevent obvious UX failure:

- standalone overdue Task: user can complete, move, or drop it
- explicit Routine occurrence from the past: record as Missed; do not Carry accumulated occurrences
- Today must not automatically receive the entire backlog

This is protective baseline behavior, not the primary product thesis.

---

## 7. Proposed Validation Focus

The old 14-day validation focused mainly on whether Reconcile helped after failure. The proposed first test should focus on whether AI creates immediate planning value.

### 7.1 Questions to test

- Does the user reach a useful first plan faster?
- Is the next action clearer after AI assistance?
- Which proposed Tasks and Routines are accepted, modified, or rejected?
- Are proposed frequencies realistic?
- Does the user execute at least one suggested action?
- Does the user return to use the generated plan?
- Does the user trust the proposal without feeling that control was taken away?

### 7.2 Suggested behavioral events

Exact taxonomy remains open, but the test will likely need events such as:

```txt
AI_PLANNING_STARTED
AI_CLARIFYING_QUESTION_ANSWERED
AI_PLAN_GENERATED
AI_PLAN_ITEM_ACCEPTED
AI_PLAN_ITEM_EDITED
AI_PLAN_ITEM_REJECTED
AI_PLAN_CONFIRMED
AI_PLAN_ABANDONED
```

Existing Task and Routine events then measure actual execution.

### 7.3 Reconcile research to preserve for Phase 2

Even if intelligent Reconcile is deferred, we should preserve the following research questions:

- What is the item-by-item review threshold before fatigue begins?
- When should the system switch to grouped or bulk review?
- Which recommendations are safe to make automatically?
- How should confidence and rationale be shown?
- When should Reconcile escalate from Task to Routine or Plan review?
- Does AI grouping outperform Structured-style manual Replan?
- Does the recovery flow improve restart behavior compared with Fresh Start?

---

## 8. Relationship to the Existing Mind Map

The existing FigJam / mind map structure does not need to be discarded. This discussion should be merged into it as a change in sequencing and responsibility.

The board currently needs these sections:

1. Product Vision
2. MVP Core Loop
3. User Flow
4. AI Responsibilities
5. AI Guardrails
6. Data Events
7. Traction Metrics
8. Current Decisions
9. Open Questions

### 8.1 Product Vision

Update the vision from a recovery-first Todo framing to:

```txt
Adaptive Planner helps a user turn an unclear intention into a small executable plan,
observe what actually happens,
and later adapt the plan without guilt or backlog collapse.
```

### 8.2 MVP Core Loop

Replace the current loop with:

```txt
User describes intention
→ AI asks only necessary clarifying questions
→ AI proposes Tasks and Routines
→ user edits and confirms
→ Today exposes the short-horizon plan
→ user acts
→ system records real behavior
→ basic recovery prevents backlog damage
```

Future loop extension:

```txt
Behavior history
→ AI-assisted Reconcile
→ grouped decisions
→ plan adaptation
```

### 8.3 User Flow

Add a new primary Day-0 flow:

```txt
Describe what you want
→ clarify constraints
→ review suggested Tasks and Routines
→ edit / accept
→ start first day
```

Keep Reconcile as a secondary future flow, not the first value moment.

### 8.4 AI Responsibilities

Split AI responsibilities into two time horizons.

#### Phase 1 — Creation assistance

- clarify intent
- propose initial Tasks
- propose initial Routines
- suggest realistic recurrence
- create a seven-day starting plan
- expose assumptions

#### Phase 2 — Recovery and adaptation

- group unresolved work
- identify similar or duplicate Tasks
- prioritize protected items
- summarize stale groups
- recommend bulk handling
- detect possible plan-level failure

### 8.5 AI Guardrails

Add:

- user confirmation before persistence
- no silent deletion or Drop
- no automatic Goal abandonment
- no hidden long-term schedule generation
- explain assumptions and grouping rationale
- high-risk domains require constrained behavior or exclusion
- reversible actions where practical
- distinguish rule-based facts from AI inference

### 8.6 Data Events

Move Reconcile events into a future/adaptation lane and add AI plan-generation evaluation events.

The board should visually distinguish:

```txt
Product behavior events
AI proposal quality events
User acceptance/edit/rejection events
Execution outcome events
```

### 8.7 Traction Metrics

Early metrics should focus on activation and plan usefulness:

- time to first confirmed plan
- percentage reaching plan confirmation
- Task acceptance rate
- Routine acceptance rate
- edit rate by proposal type
- first-action completion
- Day-1 and Day-3 return to generated plan
- user-rated clarity of next step

Reconcile retention metrics remain later-stage metrics after enough users naturally create execution history.

### 8.8 Current Decisions

Record these as proposed decisions pending resolution:

```txt
Routine is a real recurring action.
Past Routine occurrences do not accumulate through Carry.
AI enters the product first through Task and Routine creation.
AI-assisted Reconcile remains part of the product direction but is deferred.
The first AI output is short-horizon and user-approved.
The existing Phase 1 specifications require formal revision if this proposal is accepted.
```

### 8.9 Open Questions

Add:

- What exact user input starts AI planning: Goal, free-form intention, or “what do you want to organize?”
- Are Goals persisted in the first AI test or used only as planning context?
- Which domains are allowed in the pilot?
- How many clarifying questions are tolerable?
- Should the AI propose only Tasks and Routines, or also a visible high-level path?
- What is the fallback when the AI output is weak?
- What acceptance threshold justifies building AI-assisted Reconcile next?
- Which existing specs are amended versus superseded?

---

## 9. Proposed Sequence

```txt
Step 1 — Resolve Discussion 010
Step 2 — Update Product Vision and MVP Core Loop in the mind map
Step 3 — Define bounded AI creation UX and output contract
Step 4 — Rewrite the validation plan around immediate planning value
Step 5 — Amend architecture, data model, API, events, and implementation plan
Step 6 — Build AI-assisted Task and Routine creation
Step 7 — Run the short pilot and collect accept/edit/reject + execution data
Step 8 — Use that data to design AI-assisted Reconcile
```

---

## 10. Decision Requested from Claude

Review this proposal against the previous Phase 0 decisions and answer the following directly:

1. Is the “empty planner” problem a more appropriate first value problem than return-time Reconcile?
2. Is the proposed AI scope sufficiently bounded to avoid becoming an unrestricted AI life coach?
3. Which claims in this discussion are unsupported or overstated?
4. Which formal documents must be reopened if this direction is accepted?
5. Should Goal remain in the first implementation, remain only as conversational context, or be removed until progress can be evaluated?
6. What is the smallest technically honest model for AI-created Tasks and Routines?
7. What validation evidence should be required before building AI-assisted Reconcile?
8. Does any part of the accepted implementation remain reusable without biasing the team toward sunk-cost continuation?

Do not defend the previous scope merely because it has already been formalized. Also do not accept the AI direction merely because it creates a stronger narrative. Evaluate which sequence creates the strongest immediate user value with the smallest testable product.
