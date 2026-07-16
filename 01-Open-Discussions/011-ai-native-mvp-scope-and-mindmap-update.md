# Discussion 011 — AI-Native MVP Scope and Mind Map Update

## Status

Open for GPT × Claude review.

This discussion records the scope that is currently agreed in principle. It is not yet a replacement for accepted ADRs and formal specs. If resolved and accepted, the affected product, data, API, validation, and implementation documents must be updated explicitly.

Related discussion:

- [[01-Open-Discussions/010-ai-first-planning-and-reconcile-roadmap]]

---

## 1. Agreed Product Direction

The first product version will be AI-native from the beginning.

We are not building:

```txt
A manual Todo app first
→ AI planning later
→ AI Reconcile in a later phase
```

The agreed first vertical slice is:

```txt
User intention / Goal
→ AI-assisted clarification
→ AI-generated Task and Routine draft
→ user review and approval
→ Today execution
→ real behavior and missed work
→ AI-assisted Reconcile
→ user-approved plan adaptation
```

The product thesis being tested is therefore not only Task management and not only recovery from overdue work.

It is:

> Help the user create a credible plan, execute it, and adapt it when reality diverges from the plan.

This is the smallest version that honestly represents the name and promise of Adaptive Planner.

---

## 2. Agreed MVP Scope

### 2.1 Goal / Intention

The user may begin with a free-form intention or Goal.

The product should help turn that intention into a small executable plan.

The current agreed minimum Goal model is intentionally flat:

```txt
Goal
- id
- title
- status: active | completed | abandoned
```

The MVP does not introduce:

- complex Goal types
- milestone trees
- outcome-measurement engines
- project hierarchies
- path/version graphs
- automated Goal scoring

Goal exists as stable context connecting Tasks, Routines, execution history, and later Reconcile decisions.

### 2.2 AI-Assisted Plan Creation

The AI may ask only clarifying questions that materially change the proposed plan.

Initial inputs may include:

- desired outcome
- current situation
- available time
- target date
- constraints
- previous attempts or completed work

The first AI draft remains intentionally small:

```txt
0–3 initial one-time Tasks
0–2 Routines
A short seven-day starting plan
Visible assumptions that need confirmation
```

Supported Routine schedules remain limited:

```txt
Daily
Selected weekdays
Selected day of month
```

The AI draft is ephemeral until approval.

```txt
AI proposes
→ user edits, accepts, or rejects
→ only approved items are persisted
```

No unapproved generated Plan, Task, or Routine becomes product truth.

### 2.3 Manual Creation

Manual Task and Routine creation remains available as a fallback.

It is not treated as a statistically clean control group unless users are randomly assigned, which is not planned for the first small pilot.

`source: manual | ai_assisted` remains useful for descriptive analysis, not causal attribution.

### 2.4 Task

The MVP supports one-time actionable work.

Minimum expected properties:

```txt
id
goalId?
title
plannedForDate?
status
source: manual | ai_assisted
carryCount
```

The exact final schema remains open and belongs in the data-model discussion.

### 2.5 Routine

Routine is a real domain concept, even if the user creates it through the same general creation surface as a Task.

Routine represents repeated behavior with a recurrence rule.

Minimum expected properties:

```txt
id
goalId?
title
recurrenceRule
status
source: manual | ai_assisted
```

Routine occurrences must not accumulate like standalone overdue Tasks.

```txt
Past Routine occurrence
→ Done or Missed
→ never Carry as accumulated debt
```

The exact occurrence-generation strategy and storage model still need formalization.

### 2.6 Today

Today remains the main execution surface.

It should show a credible short-horizon plan, not automatically absorb all overdue work.

The MVP must preserve:

- simple completion
- clear Task vs Routine behavior
- no automatic backlog avalanche
- responsive and understandable empty/loading/error states

### 2.7 AI-Assisted Reconcile

AI-assisted Reconcile is included in the initial version.

Its purpose is not to autonomously clean the user’s life. Its purpose is to reduce a large or messy backlog into fewer, clearer, explainable decisions.

The intended pipeline is:

```txt
Rules establish known facts
→ structured grouping handles obvious relations
→ AI interprets ambiguous meaning
→ user approves consequential actions
```

AI may:

- separate standalone Tasks from Routine occurrences
- group semantically similar Tasks
- identify likely duplicates
- summarize groups of unresolved work
- surface deadline-sensitive or protected items
- rank which groups need review first
- suggest bulk actions
- identify repeated Carry or Missed patterns
- suggest reviewing a Routine, Goal, or current plan when task-level cleanup is insufficient
- explain why a recommendation was made

AI may not silently:

- Drop important Tasks
- abandon a Goal
- delete history
- rewrite a future schedule
- change a Routine cadence
- execute high-impact bulk actions

The interaction rule is:

```txt
AI groups and recommends
→ user reviews and confirms
→ system applies changes
```

### 2.8 Minimal Adaptive Replanning

The first version may suggest small plan changes after Reconcile, such as:

- keep fewer active Tasks
- reduce a Routine frequency
- move unresolved work to backlog
- choose a smaller next action
- review whether the current Goal is still active

The MVP does not attempt full autonomous long-term replanning.

---

## 3. Explicit AI Guardrails

The following are currently agreed:

- AI output is a proposal, not an authoritative decision.
- User approval is required before persistence or consequential mutation.
- No silent deletion, Drop, Goal abandonment, or future calendar rewrite.
- AI must expose relevant assumptions and grouping rationale.
- Rule-based facts must be distinguished from AI inference.
- Actions should be reversible where practical.
- Confidence should be communicated qualitatively, not with fabricated precision.
- The first pilot uses a hard domain allowlist rather than “guardrails later.”

Proposed initial allowlist:

```txt
Learning
Study
Career skills
Personal software projects
Reading
General fitness habits
```

Everything outside the allowlist should fall back to manual entry or a restricted response until the policy is defined.

---

## 4. Validation Scope

The first pilot tests two formal hypotheses in the same product.

Neither test is causal. Both use absolute behavior rates and predeclared thresholds.

### H1 — AI-Assisted Creation

> Users can reach a useful first plan through bounded AI-assisted creation.

Candidate metrics:

- AI planning started
- plan generated
- plan confirmation rate
- Task acceptance rate
- Routine acceptance rate
- edit rate
- rejection rate
- flow abandonment
- time to first confirmed plan
- first action completion
- Day-1 and Day-3 return
- user-rated clarity of next step

### H2 — AI-Assisted Reconcile

> AI can reduce unresolved work into a smaller set of understandable, user-approved decisions.

Candidate metrics:

- Reconcile shown
- Reconcile started
- Reconcile completed
- Reconcile skipped
- suggestion acceptance rate
- suggestion edit/rejection rate
- grouping correction rate
- unresolved items before and after
- flow abandonment by backlog size
- meaningful action after Reconcile

Reconcile results must be segmented by context rather than collapsed into one completion rate.

Proposed context fields:

```txt
absenceDays
overdueTaskCount
oldestTaskAgeDays
routineOccurrenceCount
```

Possible analysis bands:

```txt
Light Reconcile
- one-day deviation
- 1–3 unresolved Tasks

Medium Reconcile
- several days or a moderate backlog

Recovery Reconcile
- meaningful absence or a large stale backlog
```

The exact thresholds remain open.

---

## 5. What Changes in the Current Mind Map

The existing board structure remains useful, but several sections must be rewritten.

### 5.1 Product Vision — Change

Current recovery-first framing should become:

```txt
Adaptive Planner turns an unclear intention into a small executable plan,
helps the user act on it,
and adapts the plan when reality diverges.
```

Reconcile becomes part of the full product loop, not the whole product thesis.

### 5.2 MVP Core Loop — Change

Replace the manual-first loop with:

```txt
Describe intention / Goal
→ AI clarification
→ review suggested Tasks and Routines
→ confirm plan
→ execute in Today
→ record behavior
→ AI-assisted Reconcile
→ approve adaptations
```

### 5.3 User Flow — Change

Add or replace the main flows with:

#### Day-0 AI Creation

```txt
Enter intention
→ answer limited clarifying questions
→ review Goal, Tasks, and Routines
→ edit / accept / reject
→ confirm
→ enter Today
```

#### Daily Execution

```txt
Open Today
→ complete Task or Routine occurrence
→ reschedule or leave unresolved when necessary
```

#### AI Reconcile

```txt
Open with unresolved work
→ view AI summary and groups
→ review high-risk items
→ approve bulk and item-level decisions
→ review suggested plan adjustment
→ return to a clean Today
```

### 5.4 AI Responsibilities — Major Change

Split the section into two equal product responsibilities.

#### Planning AI

- clarify intention
- expose assumptions
- propose Tasks and Routines
- propose realistic cadence
- produce a short seven-day plan

#### Reconcile AI

- classify ambiguous unresolved work
- group similar Tasks
- summarize backlog
- suggest review priority
- propose bulk handling
- escalate to Routine or Goal review

### 5.5 AI Guardrails — Change

Add the agreed rules from Section 3.

The board should visually distinguish:

```txt
Rule-based decision
AI inference
User-owned decision
```

### 5.6 Data Events — Change

Add AI proposal-quality and decision events alongside execution events.

Candidate additions:

```txt
AI_PLANNING_STARTED
AI_PLAN_GENERATED
AI_PLAN_ITEM_ACCEPTED
AI_PLAN_ITEM_EDITED
AI_PLAN_ITEM_REJECTED
AI_PLAN_CONFIRMED
AI_PLAN_ABANDONED

AI_RECONCILE_GENERATED
AI_RECONCILE_GROUP_ACCEPTED
AI_RECONCILE_GROUP_EDITED
AI_RECONCILE_GROUP_REJECTED
AI_RECONCILE_COMPLETED
```

The exact names and payloads remain open.

### 5.7 Traction Metrics — Change

Activation metrics must now include:

- time to first useful plan
- plan confirmation
- accepted and edited AI suggestions
- first completed action

Continuity and recovery metrics must include:

- Reconcile completion by severity
- backlog reduction
- suggestion correction
- return to meaningful action

### 5.8 Current Decisions — Change

Add the decisions listed in Section 7 below.

Remove or mark as superseded any statement that Phase 1 excludes:

- AI runtime
- Routine
- AI-assisted Reconcile

### 5.9 Open Questions — Replace / Expand

Use Section 8 of this discussion as the current question list.

---

## 6. What Remains Stable in the Current Mind Map

The following product and project principles remain valid unless a later discussion explicitly changes them.

### Product Principles

- Planner is the initial product entry point.
- The system should learn from real behavior rather than personality questionnaires.
- User agency remains central.
- Avoid guilt-heavy overdue UX and punitive streak design.
- Today should remain credible and limited.
- AI must not create invisible or irreversible decisions.
- Behavioral evidence matters more than self-report alone.

### Team and Collaboration

The board is still shared by:

```txt
Mahdi — product + frontend + coordination
Backend developer — Java / Spring / PostgreSQL
Designer — UX/UI and interaction states
```

### Technical Direction

The following remain directionally stable:

- React frontend
- Java / Spring Boot backend
- PostgreSQL
- API-first frontend/backend boundary
- PWA direction
- offline-first deferred beyond the initial version
- behavioral event logging
- timezone-aware dates
- explicit validation and database invariants

Exact AI provider, model, prompt architecture, and runtime integration are not yet decided.

### Research Discipline

- Direct sources outrank unattributed summaries.
- Shipped competitor features prove precedent, not efficacy.
- Untraceable social quotes are not treated as verified evidence.
- Small pilot results produce directional signals, not causal proof.
- Inconclusive remains a valid outcome.

### Board Structure

The nine top-level mind map sections remain:

1. Product Vision
2. MVP Core Loop
3. User Flow
4. AI Responsibilities
5. AI Guardrails
6. Data Events
7. Traction Metrics
8. Current Decisions
9. Open Questions

The structure remains; the content and sequencing change.

---

## 7. Decisions Currently Agreed in Principle

These are not yet formal ADRs, but they represent the current team/product agreement to be reviewed:

```txt
1. The first product version is AI-native.
2. AI-assisted Goal/Task/Routine creation is in the initial scope.
3. AI-assisted Reconcile is also in the initial scope.
4. The MVP tests the full Plan → Execute → Adapt loop.
5. Goal remains simple and persisted.
6. Routine is a separate domain model from Task.
7. Past Routine occurrences become Done or Missed, not Carried.
8. AI drafts remain ephemeral until user approval.
9. Manual creation remains available as fallback.
10. AI may group and recommend but may not silently apply consequential actions.
11. The first plan is short-horizon and intentionally small.
12. The pilot uses a hard domain allowlist.
13. Creation and Reconcile are both formal validation hypotheses.
14. Reconcile must be analyzed by backlog severity and absence context.
15. Existing technical foundations should be reused where they remain direction-independent.
```

---

## 8. Open Questions

### Product and UX

1. What exact input starts the experience: Goal, free-form intention, or both?
2. Does the user see Goal as a primary UI concept, or only as context behind Tasks and Routines?
3. How many clarifying questions are acceptable before the user experiences friction?
4. Should AI ask questions conversationally or through structured controls?
5. What happens when the AI output is low-quality or the user rejects everything?
6. Should the user be able to regenerate the full draft or only edit individual items?
7. How should AI assumptions be shown without making onboarding feel like a legal contract?
8. What is the exact Day-0 transition from approved plan to Today?
9. What happens when the user has multiple active Goals?
10. What is the smallest useful representation of a high-level plan without introducing Project/Path complexity?

### Task and Routine Domain

11. Is Routine created through the same form as Task with a Repeat field, or through a visibly distinct flow?
12. How are Routine occurrences generated: on open, scheduled materialization, or another method?
13. How much occurrence history is persisted?
14. Can a Routine be paused retroactively for illness, travel, or vacation?
15. How are edits to a Routine applied to past and future occurrences?
16. What happens when a Routine is linked to a completed or abandoned Goal?
17. Is `source` stored directly on Task/Routine or only in immutable event metadata?

### AI Planning

18. Which exact domains are in the first allowlist?
19. What model/provider is used, and what is the fallback when it is unavailable?
20. What structured output contract should the AI return?
21. What validation rejects unsafe, malformed, or oversized output?
22. How are duplicate or contradictory suggestions handled before user review?
23. Should the AI propose dates directly or propose priorities that the user schedules?
24. How do we evaluate plan quality beyond acceptance rate?
25. What prompt/evaluation dataset is needed before the pilot?

### AI Reconcile

26. What exact trigger opens AI Reconcile?
27. Is Reconcile mandatory before Today, dismissible, or accessible as a separate action?
28. How do we prevent Reconcile itself from becoming a long blocking flow?
29. What backlog size triggers grouping instead of item-by-item review?
30. Which decisions can be proposed in bulk?
31. Which decisions always require individual review?
32. How are high-risk or deadline-sensitive Tasks protected?
33. How is AI grouping rationale displayed?
34. How can the user correct a wrong group?
35. What qualifies as a duplicate versus related but distinct Tasks?
36. When should Reconcile escalate from Task review to Routine review?
37. When should Reconcile surface the Goal itself?
38. Can Reconcile recommend lowering Routine frequency in the MVP?
39. What does “Drop” mean technically: archive, terminal status, or removal from active plan?
40. What actions are reversible, and for how long?

### Data and Metrics

41. What is the final AI event taxonomy?
42. What metadata is necessary to evaluate proposal quality without storing sensitive prompt content unnecessarily?
43. What are the predeclared Pass / Weak Pass / Fail / Inconclusive thresholds for H1?
44. What are the thresholds for H2?
45. How are Light, Medium, and Recovery Reconcile defined?
46. How do we distinguish weak AI output from weak Today UX or low user motivation?
47. What qualitative research accompanies the event data?
48. What privacy and retention rules apply to AI prompts, outputs, and rationales?

### Architecture and Implementation

49. Which existing ADR becomes superseded?
50. Which existing specs are amended versus fully replaced?
51. Do AI creation and AI Reconcile use one backend orchestration service or separate bounded modules?
52. Is AI invoked synchronously, streamed, or through a job model?
53. What timeout and retry behavior is acceptable?
54. How is idempotency handled when approved AI proposals create several entities?
55. What happens when AI generation succeeds but persistence partially fails?
56. How are prompt versions and model versions recorded?
57. Is an explicit AI proposal entity needed for auditability, or are events plus ephemeral frontend state sufficient?
58. How will testing work without depending on live model calls?
59. What is the minimal evaluation harness required before implementation?
60. How does the app behave during internet or AI-provider outages?

---

## 9. Documents That Must Be Reopened if Discussion 011 Is Accepted

At minimum:

- `README.md`
- `00-START-HERE.md`
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]
- [[02-Decisions/ADR-003-phase-0-closure]]
- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/api-contracts-phase-1]]
- [[04-Specs/phase-1-implementation-stack]] where AI runtime assumptions are affected
- validation plan
- metrics definitions
- [[05-Implementation/phase-1-plan]]

Auth remains mostly independent and should only change if AI-specific authorization or privacy requirements create a concrete need.

---

## 10. Requested Review from Claude

Review the agreed scope without reopening the already settled question of whether AI belongs in the first version.

The remaining review task is to make the scope technically honest and bounded.

Please answer:

1. Which item in the agreed MVP is still too broad to be implementable as a first vertical slice?
2. Which open questions must be resolved before updating the mind map?
3. Which open questions can safely wait until formal technical design?
4. Is the Goal model still sufficient once AI Reconcile may escalate to Goal review?
5. Is a separate RoutineOccurrence model required in the initial version?
6. What is the smallest safe structured-output contract for Planning AI?
7. What is the smallest safe structured-output contract for Reconcile AI?
8. Which Reconcile actions should be rule-based rather than AI-generated?
9. What minimal audit trail is required for AI proposals and user decisions?
10. Which current specs can be amended instead of rewritten?

Do not argue for removing AI from the initial version. Challenge scope, boundaries, data requirements, and sequence within the agreed AI-native vertical slice.
