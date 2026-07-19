# Discussion 017 — AI Reconcile Intelligence and Actions

## Status

Open for GPT × Claude review.

This revision defines a bounded, metric-driven Reconcile intelligence model. AI may not infer freely, diagnose the user, or invent recommendations. It may only evaluate approved metrics against accepted rules, explain the matched evidence, and present predefined user-approved actions.

Related accepted discussions:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/016-reconcile-trigger-and-severity]]

---

## 1. Scope

Discussion 016 determines **when** Reconcile appears and **how strongly** it is presented.

Discussion 017 determines what happens after Reconcile is opened:

- which facts are handled deterministically
- which metrics may be evaluated
- which bounded rules may match
- how items are grouped
- which recommendation types are allowed
- which actions remain user-owned
- when a Task-level pattern may justify a Routine or Project-level suggestion
- how rationale and evidence are shown
- what Reconcile may pass to Adaptive Planning

Core question:

```txt
Given accepted execution facts,
which limited and explainable actions may the product suggest,
without guessing about user intent or silently rewriting commitments?
```

---

## 2. Explicitly Out of Scope

This discussion does not define:

- Reconcile trigger or severity — Discussion 016
- open-ended semantic interpretation
- personality, motivation, discipline, or psychological diagnosis
- unrestricted duplicate detection by language similarity
- automatic Goal achievement or abandonment
- automatic Project completion or stopping
- automatic Routine stopping or continuation creation
- database schema, indexes, or event persistence — Discussion 019
- provider, prompt, API, or runtime architecture — Discussion 020
- exact validation plan — Discussion 021
- implementation sequencing — Discussion 022
- full Adaptive Planning algorithm or weekly-plan generation
- visual styling, animation, or responsive layout

The MVP intentionally prefers a small number of explicit rules over broad AI coverage.

---

## 3. Correct Accepted Product Hierarchy

The original skeleton incorrectly stated that Task and Routine never link directly to Goal.

Accepted ownership from Discussion 012 is:

```txt
Project → one Goal or standalone
Task → one Goal, one Project, or standalone
Routine → one Goal, one Project, or standalone
RoutineOccurrence → exactly one Routine
```

Task and Routine may not belong to Goal and Project simultaneously.

Reconcile must therefore support:

- Project-owned Tasks and Routines
- direct Goal-owned Tasks and Routines
- standalone Tasks and Routines

Stable ownership is always used before any secondary grouping.

---

## 4. Governing Principle: No Free Interpretation

Accepted direction from Mahdi:

> AI must not guess, even in semantic analysis. The approved metrics, rule combinations, and allowed recommendation types must be explicit.

The governing pipeline is:

```txt
deterministic execution facts
→ stable ownership grouping
→ approved metric calculation
→ bounded rule matching
→ predefined recommendation options
→ explicit user decision
```

AI is not an open-ended behavioral analyst.

For MVP, its allowed role is:

```txt
structured evaluator
+ explanation generator
+ constrained recommendation presenter
```

It may not:

- invent a new problem category
- infer that the user lacks motivation
- infer that a Goal is no longer meaningful
- infer low capacity from absence
- infer failure from raw MISSED counts
- recommend an action outside the accepted rule action set
- change canonical entities without confirmation

---

## 5. Deterministic Rule and AI Boundary

### 5.1 Facts Established Without AI

The following are deterministic facts:

- Task ownership and parent status
- Task `plannedDate`
- Task age in local calendar days
- Task Carry count
- Task lifecycle state
- whether a Task is overdue
- whether a Task is protected
- whether a deadline exists
- whether a deadline risk rule matches
- Routine ownership and lifecycle
- occurrence scheduled local date
- occurrence DONE or MISSED status
- observed occurrence count
- missed count within an observed window
- absence-contamination level
- recurrence edit count
- Project and Goal lifecycle state
- active child counts
- structural conflicts
- Reconcile severity and evaluated boundary from Discussion 016

AI must consume these facts rather than recreate them from prose.

### 5.2 Bounded AI Responsibilities

AI may:

- turn matched rule facts into concise human-readable rationale
- present only the recommendation options attached to the matched rule
- explain consequences already established by product rules
- summarize a resolved Reconcile session
- ask one targeted clarification when an accepted rule requires user context

AI may not select a recommendation type before a deterministic rule match exists.

### 5.3 Recommendation Contract

Every AI-backed suggestion must include:

```txt
- matchedRuleId
- affectedEntityIds
- observedMetrics
- exact problem statement
- allowedActions[]
- consequence summary
- evidence quality
```

The user-facing explanation must remain understandable without exposing implementation jargon.

---

## 6. Grouping Model

Grouping is deterministic first.

### 6.1 First Boundary: Stable Ownership

Items are first grouped by canonical ownership:

```txt
Project-owned work → under that Project
Goal-owned work → under that Goal
standalone work → standalone section
RoutineOccurrences → under their Routine
```

AI may not flatten Project-owned work into direct Goal work.

### 6.2 Second Boundary: Rule Relationship

Within an ownership group, items may be grouped by a matched rule:

- repeated Carry on the same Task
- old unresolved Tasks
- common deadline risk
- same Routine execution pattern
- same structural conflict
- same correction review

### 6.3 No Free Semantic Grouping in MVP

The MVP does not allow AI to group items solely because titles appear similar.

Examples excluded from MVP:

- likely duplicates inferred from wording alone
- grouping by broad topic similarity
- grouping by inferred intent
- grouping by assumed shared outcome

A future discussion may add duplicate or similarity rules only after explicit metrics and validation requirements exist.

### 6.4 User Control

The user may:

- remove an item from a proposed rule group
- review an item individually
- reject the recommendation for the whole group
- apply an allowed action only to selected items

MVP does not require arbitrary user-created merge groups inside Reconcile.

---

## 7. Protected Items

A protected item must not receive destructive or de-prioritizing bulk suggestions.

A Task is protected when one or more accepted conditions are true:

- user explicitly marked it protected
- hard deadline exists
- legal, financial, health, or external obligation flag exists
- it blocks another accepted dependency
- it belongs to an active parent marked high priority
- user recently reaffirmed it
- it is currently in progress

Exact field representation is deferred to Discussion 019.

Protected-item rules:

- no automatic or bulk Drop suggestion
- no silent Backlog move
- no suggestion that hides deadline risk
- rationale must identify the protection reason
- available actions emphasize scheduling, splitting, or direct review

---

## 8. Initial MVP Intelligence Rules

The MVP should begin with a small accepted rule catalog.

### Rule R1 — Repeated Carry Task

Trigger metrics:

```txt
Task.status = ACTIVE
carryCount >= 2
```

Problem statement:

> This Task has been moved more than once and may need a clearer decision.

Allowed actions:

```txt
- REPLAN_TO_EXPLICIT_DATE
- MOVE_TO_BACKLOG
- SPLIT_TASK
- DROP_TASK, only when not protected and no deadline risk
- KEEP_UNCHANGED
```

Constraints:

- first Carry alone does not trigger this rule
- the rule applies to the same Task identity
- it does not diagnose low motivation or overload
- matching this rule does not automatically change Project or Goal state

### Rule R2 — Old Unresolved Task

Proposed trigger metrics:

```txt
Task.status = ACTIVE
ageDays >= staleAgeThreshold
no blocking structural conflict
```

Additional destructive-action constraints:

```txt
DROP_TASK allowed only if:
protected = false
AND deadlineRisk = false
```

Allowed actions:

```txt
- REPLAN_TO_EXPLICIT_DATE
- MOVE_TO_BACKLOG
- DROP_TASK, when allowed
- KEEP_UNCHANGED
```

Open threshold proposal:

```txt
staleAgeThreshold = 7 local days
```

The age threshold identifies a review candidate, not proof that the Task is unimportant.

### Rule R3 — Deadline Risk

Trigger metrics must be deterministic:

```txt
deadline exists
Task is unresolved
remainingExecutionWindow <= accepted risk threshold
```

Allowed actions:

```txt
- SCHEDULE_NEXT_ACTION
- SPLIT_TASK
- REVIEW_CONFLICTING_TASKS
- KEEP_WITH_ACKNOWLEDGED_RISK
```

Constraints:

- no automatic Drop recommendation
- no Backlog recommendation that would conceal the deadline
- explanation must show the real deadline and remaining time
- a same-day newly detected deadline risk may use the retrigger exception accepted in Discussion 016

Exact risk thresholds remain open for later validation, but the rule must be deterministic before release.

### Rule R4 — Routine Mismatch Candidate

This rule requires enough observed execution and must exclude heavily absence-contaminated periods.

Proposed trigger metrics:

```txt
observedOccurrenceCount >= minimumObservedOccurrences
missedRatio >= missedRatioThreshold
absenceContamination <= acceptedMaximum
Routine.status = ACTIVE
```

Allowed actions:

```txt
- CHANGE_ROUTINE_DAYS
- REDUCE_ROUTINE_FREQUENCY
- REVIEW_ROUTINE_TIME
- STOP_ROUTINE
- KEEP_ROUTINE_UNCHANGED
```

Constraints:

- AI does not claim the Routine is incompatible with the user's lifestyle
- raw missed count alone is insufficient
- automatically reconstructed absence facts have lower evidence quality
- stopping requires explicit user confirmation
- if the user later resumes, Discussion 015 requires a new continuation Routine

Current proposals for Claude review:

```txt
minimumObservedOccurrences = 6
missedRatioThreshold = 0.5
```

The exact absence-contamination boundary remains open.

### Rule R5 — Project Execution Overload Candidate

This rule is intentionally narrow.

Proposed trigger metrics:

```txt
activeTaskCount >= minimumProjectTaskCount
repeatedCarryTaskCount >= minimumRepeatedCarryTasks
observed across at least minimumObservedPeriods
Project.status = ACTIVE
```

Allowed actions:

```txt
- MOVE_SELECTED_TASKS_TO_BACKLOG
- SPLIT_SELECTED_TASKS
- EXTEND_PROJECT_TARGET_DATE
- KEEP_PROJECT_UNCHANGED
```

Constraints:

- no generic “review your Project” suggestion
- AI must state the exact metric problem
- no automatic Project completion
- no automatic Project stop
- no child detachment
- no Goal-level conclusion

Current proposals:

```txt
minimumProjectTaskCount = 4
minimumRepeatedCarryTasks = 2
minimumObservedPeriods = 2
```

### Rule R6 — Structural Lifecycle Conflict

Examples:

- Project completion attempted with active child Tasks
- Goal terminal transition attempted with active children

This is not an AI discovery rule. The conflict is deterministic.

Allowed Reconcile behavior:

- explain the exact conflicting children
- present only lifecycle actions accepted in Discussion 015
- show the consequence of each action

For Project active Tasks, allowed options include:

```txt
- COMPLETE_SELECTED_TASKS
- DROP_SELECTED_TASKS
- DETACH_OR_REPARENT_SELECTED_TASKS
- CANCEL_PARENT_TRANSITION
```

AI may explain but must not select on behalf of the user.

---

## 9. Action Taxonomy

### 9.1 Task Actions

```txt
COMPLETE_TASK
KEEP_UNCHANGED
REPLAN_TO_EXPLICIT_DATE
MOVE_TO_BACKLOG
DROP_TASK
SPLIT_TASK
```

Task actions must preserve the execution rules in Discussion 015.

### 9.2 RoutineOccurrence Actions

```txt
MARK_DONE
KEEP_MISSED
CORRECT_OCCURRENCE
ADD_OPTIONAL_CONTEXT
```

Occurrence restrictions:

- no Carry
- no Replan of the historical occurrence
- scheduled local date does not change

### 9.3 Routine Actions

```txt
CHANGE_ROUTINE_DAYS
REDUCE_ROUTINE_FREQUENCY
REVIEW_ROUTINE_TIME
STOP_ROUTINE
KEEP_ROUTINE_UNCHANGED
```

Routine edits must follow prospective effective-date rules from Discussion 015.

### 9.4 Project Actions

For MVP, Reconcile may suggest limited structural changes:

```txt
MOVE_SELECTED_TASKS_TO_BACKLOG
SPLIT_SELECTED_TASKS
EXTEND_PROJECT_TARGET_DATE
KEEP_PROJECT_UNCHANGED
```

Project terminal actions are not normal AI recommendations.

When the user independently initiates a Project terminal transition, Reconcile may support the required child-resolution flow from Discussion 015.

### 9.5 Goal Actions

MVP Reconcile does not recommend:

```txt
ACHIEVE_GOAL
ABANDON_GOAL
```

It also does not infer that completed or stopped Projects prove Goal outcome achievement.

When a Goal has no active Project, AI may only report the deterministic state:

> This Goal currently has no active Project.

Whether to create a new Project, achieve the Goal, or abandon it remains a user-owned planning decision outside normal Reconcile recommendation rules.

---

## 10. Bulk Suggestions

### 10.1 Bulk Eligibility Principle

A bulk suggestion is allowed only when:

```txt
- every selected item matches the same accepted rule
- every selected item supports the same allowed action
- consequences are local and visible
- protected items are excluded when required
- the user previews the selected items
- the user explicitly confirms
```

### 10.2 Proposed Bulk-Safe Actions

```txt
- MOVE_SELECTED_TASKS_TO_BACKLOG
- REPLAN_SELECTED_TASKS_TO_ONE_EXPLICIT_DATE
- KEEP_SELECTED_ITEMS
```

`DROP_SELECTED_TASKS` may be allowed only when every item is unprotected, has no deadline risk, and is individually visible before confirmation.

### 10.3 Item-Level Review Required

The following require item-level confirmation:

```txt
- COMPLETE_TASK
- SPLIT_TASK
- DROP_TASK when any contextual risk exists
- CORRECT_OCCURRENCE
- STOP_ROUTINE
- DETACH_OR_REPARENT_TASK
- any Project or Goal terminal decision
```

### 10.4 Forbidden Autonomous Actions

AI must never autonomously:

- complete work
- drop work
- stop a Routine
- complete or stop a Project
- achieve or abandon a Goal
- merge or delete entities
- detach or reparent children
- change deadlines

---

## 11. Escalation Model

Escalation means moving from an item-level rule to a bounded parent-level rule. It does not mean issuing a vague “review this” recommendation.

### 11.1 Task to Routine

Allowed only when Routine rule R4 matches.

The product must state:

- observed occurrence count
- missed ratio
- absence contamination
- exact allowed Routine changes

### 11.2 Tasks to Project

Allowed only when Project rule R5 matches.

The product must state an exact problem such as:

> Two of five active Tasks in this Project have each been carried at least twice across two observed periods.

It must not merely say:

> Consider reviewing this Project.

### 11.3 Project to Goal

MVP does not perform interpretive Goal escalation.

The product may report deterministic Goal context, but may not infer:

- Goal no longer matters
- Goal has been achieved
- Goal should be abandoned
- Project difficulty proves Goal mismatch

Goal decisions remain user-owned and may later receive a separate bounded rule set.

### 11.4 Standalone Work

Standalone Tasks and Routines use the same Task and Routine rules without parent escalation.

The absence of a parent must not cause AI to invent one or infer hidden Goal context.

---

## 12. Recommendation Frequency and Suppression

Discussion 016 limits primary Reconcile presentation. Discussion 017 additionally limits recommendation volume inside a session.

Accepted direction:

- AI should not produce a recommendation for every eligible item
- deterministic quick actions are sufficient for simple cases
- AI-backed recommendations require enough evidence and a matched rule
- multiple matches for the same entity should be consolidated

Proposed MVP limits:

```txt
LIGHT session:
- at most 1 AI-backed recommendation group

MEDIUM session:
- at most 3 AI-backed recommendation groups

RECOVERY session:
- present recommendations in chunks of at most 3 groups at a time
```

Within one group:

- show one problem statement
- show at most 4 allowed actions
- avoid repeating the same rationale per item

Claude should review whether these limits belong in the product contract or later UX specification.

---

## 13. Evidence Quality and Rationale

### 13.1 No Artificial Probability

MVP does not show fabricated numerical confidence percentages.

Conceptual evidence quality may be:

```txt
SUFFICIENT
LIMITED
ABSENCE_CONTAMINATED
```

A rule may generate an action recommendation only when evidence quality satisfies that rule's minimum.

### 13.2 Required Rationale

Every recommendation must answer:

```txt
What exact pattern matched?
Which facts support it?
Why are these actions allowed?
What consequence follows from each consequential action?
```

### 13.3 Prohibited Rationale

Do not use claims such as:

- you seem unmotivated
- this Goal no longer fits you
- your productivity is low
- this Routine does not match your lifestyle
- you are taking on too much, unless a specific accepted workload rule matched

---

## 14. Consequence Disclosure

Before confirmation, consequential actions must disclose deterministic effects.

### Stop Routine

Must explain:

- future occurrences stop after the accepted boundary
- historical occurrences remain
- later Resume creates a new continuation Routine

### Complete or Stop Project

When the user initiated the transition, disclose:

- active child Tasks requiring resolution
- active Project-owned Routines that will stop
- existing same-day occurrence behavior
- future occurrence ineligibility
- no active child may remain under the terminal Project

### Goal Terminal Actions

If initiated outside normal Reconcile recommendation rules, disclose:

- active Projects, Tasks, and Routines requiring resolution
- Goal outcome must be confirmed by the user
- execution completion does not prove real-world achievement

---

## 15. Minimum Adaptive Planning Handoff

Reconcile and Adaptive Planning remain separate.

Reconcile may output structured resolved evidence:

```txt
- Tasks completed
- Tasks carried or replanned
- Tasks moved to Backlog
- Tasks dropped
- Routine changes confirmed
- recommendations rejected
- evidence quality
- absence-contaminated periods
```

Allowed follow-up:

- record the evidence
- summarize the session
- offer entry into a separate planning review

Not allowed inside Reconcile:

- silently regenerate the weekly plan
- change future workload without approval
- infer long-term user capacity from one session
- treat absence as confirmed failure

---

## 16. Scenario Checks

### Scenario A — One Overdue Task

```txt
ageDays = 1
carryCount = 0
no deadline risk
```

Expected:

- deterministic quick actions only
- no AI-backed interpretation required
- user may complete, replan, backlog, drop, or keep as allowed

### Scenario B — Task Carried Twice

Expected:

- R1 matches
- exact Carry count shown
- limited allowed actions shown
- no claim about motivation or capacity

### Scenario C — Protected Old Task

```txt
ageDays = 12
hard deadline exists
```

Expected:

- R2 may identify stale age
- destructive options excluded
- R3 deadline protection dominates available actions
- schedule, split, or conflict review offered

### Scenario D — Routine During Absence

```txt
10 reconstructed MISSED occurrences
little observed interaction
```

Expected:

- no Routine mismatch recommendation from raw MISSED count
- evidence marked absence-contaminated
- user may correct history
- no frequency reduction recommendation until R4 minimum evidence is satisfied

### Scenario E — Observed Routine Pattern

```txt
8 observed occurrences
5 MISSED
low absence contamination
```

Expected proposal:

- R4 matches
- exact ratio shown
- only predefined Routine actions offered
- no lifestyle inference

### Scenario F — Project Carry Pattern

```txt
5 active Tasks
2 Tasks carried at least twice
pattern observed across 2 periods
```

Expected proposal:

- R5 matches
- exact affected Tasks and periods shown
- limited actions: backlog selected Tasks, split selected Tasks, extend target date, or keep
- no Stop Project recommendation

### Scenario G — Completed Projects Under Goal

Expected:

- AI does not infer Goal achievement
- product may report no active Project remains
- user owns the decision to create a new Project, achieve, or abandon the Goal

### Scenario H — Standalone Task

Expected:

- Task rules work without parent context
- no Project or Goal is invented
- no parent-level escalation occurs

---

## 17. Stabilized Decisions

The following are current accepted directions unless Claude finds a contradiction:

- AI may not perform free semantic interpretation
- accepted metrics and rules must be explicit
- recommendations must come from predefined action sets
- rules establish facts before AI explains them
- ownership is the first grouping boundary
- direct Goal-owned work is valid
- standalone work is supported
- semantic similarity and duplicate detection are excluded from MVP
- protected items cannot receive unsafe destructive suggestions
- one Carry is ordinary; repeated Carry begins at two
- raw MISSED count does not establish Routine mismatch
- absence-contaminated periods reduce evidence quality
- Project escalation must state a precise metric problem
- vague “review this Project” recommendations are forbidden
- normal Reconcile does not recommend Project stop or completion
- normal Reconcile does not recommend Goal achievement or abandonment
- Project completion does not prove Goal achievement
- consequential actions require explicit user confirmation
- Reconcile may pass structured evidence to Adaptive Planning but does not rebuild the plan

---

## 18. Open Product Questions and Current Proposals

### Q1. Is the initial MVP rule catalog small enough?

Current proposal: launch only R1–R6.

Claude should review whether any rule is still too broad or whether one essential rule is missing.

### Q2. What is the stale Task threshold?

Current proposal:

```txt
staleAgeThreshold = 7 local days
```

The threshold triggers review eligibility, not automatic Drop.

### Q3. What minimum evidence should Routine rule R4 require?

Current proposal:

```txt
minimumObservedOccurrences = 6
missedRatioThreshold = 0.5
```

Open: exact allowed absence-contamination level.

### Q4. Are Project rule R5 thresholds coherent?

Current proposal:

```txt
minimumProjectTaskCount = 4
minimumRepeatedCarryTasks = 2
minimumObservedPeriods = 2
```

Claude should test edge cases where a small Project is still clearly blocked.

### Q5. Should AI recommendation-group limits be part of the product contract?

Current proposal:

```txt
LIGHT: 1 group
MEDIUM: 3 groups
RECOVERY: chunks of 3 groups
```

Alternative: defer exact counts to UX while preserving “limited and chunked.”

### Q6. Is `DROP_SELECTED_TASKS` safe as a bulk action?

Current proposal: only when every item is unprotected, has no deadline risk, is fully previewed, and the user explicitly confirms.

Alternative: require item-level confirmation for all Drop actions in MVP.

### Q7. Should `EXTEND_PROJECT_TARGET_DATE` be allowed from Reconcile?

Current proposal: yes when R5 matches, but only with explicit new date and user confirmation.

Claude should verify this does not silently become Adaptive Planning.

### Q8. Should Goal no-active-Project context provide navigation choices?

Current proposal: show deterministic context and links to separate planning actions, but do not frame them as AI recommendations.

### Q9. Is `SPLIT_TASK` sufficiently defined for MVP?

Current proposal: Reconcile may recommend splitting, but the user must define or approve resulting Tasks. Exact split-generation contract may need a later discussion.

### Q10. Should evidence quality use three labels or remain hidden?

Current proposal:

```txt
SUFFICIENT
LIMITED
ABSENCE_CONTAMINATED
```

The labels may guide product behavior even if not shown verbatim to the user.

---

## 19. Mind Map Impact — Record Only, Do Not Apply Yet

After consolidation, record the following.

### Product Vision

Add:

- AI organizes evidence but does not guess about the user
- Reconcile suggestions are bounded, explainable, and user-controlled
- simplicity and trust take priority over broad AI coverage in MVP
- exact problems and limited actions replace vague advice

### MVP Core Loop

```txt
Open Reconcile
→ receive deterministic ownership groups
→ calculate approved metrics
→ match bounded rules
→ show exact evidence and limited actions
→ user confirms, rejects, or edits
→ execution history updates
→ structured evidence may feed later Adaptive Planning
```

### User Flow

Add:

```txt
simple eligible item
→ deterministic quick actions

matched intelligence rule
→ exact problem statement
→ observed metrics
→ limited allowed actions
→ user confirmation

consequential parent action
→ consequence disclosure
→ item-level child resolution
→ explicit confirmation
```

### AI Responsibilities

Add:

- explain matched deterministic rules
- present only accepted recommendation types
- summarize observed metrics
- consolidate repeated recommendations
- ask only rule-required targeted clarification
- create structured Reconcile summary for later planning evidence

Remove:

- open-ended semantic interpretation
- free duplicate detection
- personality or motivation inference
- vague Project or Goal review advice

### AI Guardrails

Add:

- no recommendation without a matched accepted rule
- no invented metric or problem type
- no destructive action for protected items
- no raw MISSED-count interpretation
- no absence-as-failure inference
- no Project stop/complete recommendation in normal MVP Reconcile
- no Goal achieve/abandon recommendation
- no autonomous lifecycle change
- no confidence percentage presented as scientific certainty
- no vague recommendation without exact evidence and allowed actions

### Data Events

Candidates for Discussion 019:

```txt
RECONCILE_RULE_EVALUATED
RECONCILE_RULE_MATCHED
RECONCILE_RECOMMENDATION_PRESENTED
RECONCILE_RECOMMENDATION_ACCEPTED
RECONCILE_RECOMMENDATION_REJECTED
RECONCILE_RECOMMENDATION_EDITED
RECONCILE_BULK_PREVIEWED
RECONCILE_BULK_CONFIRMED
RECONCILE_PROTECTED_ITEM_EXCLUDED
RECONCILE_PARENT_ESCALATION_PRESENTED
RECONCILE_ADAPTIVE_HANDOFF_CREATED
```

Each rule event should later preserve:

- rule ID and version
- input metrics
- affected entities
- evidence quality
- allowed actions
- selected action

### Traction Metrics

Valid metrics:

- recommendation acceptance by rule ID
- recommendation rejection by rule ID
- percentage of matched rules resolved without AI text generation
- user edits before confirmation
- bulk-preview cancellation rate
- protected-item exclusion correctness
- subsequent repeated Carry after R1 resolution
- Routine change acceptance after R4
- return-to-execution after Reconcile

Guardrail metrics:

- unsupported recommendation rate, expected zero
- recommendation shown without matched rule, expected zero
- destructive suggestion involving protected item, expected zero
- Project or Goal terminal recommendation rate in MVP, expected zero
- absence-contaminated recommendation rate
- excessive recommendations per session

### Current Decisions

Record:

- deterministic facts precede AI explanation
- ownership grouping precedes rule grouping
- direct Goal-owned and standalone work are supported
- MVP excludes free semantic grouping and duplicate detection
- AI uses only accepted metrics and bounded rules
- recommendations use predefined action sets
- protected items restrict destructive actions
- exact evidence must accompany every suggestion
- Goal outcome remains user-confirmed
- Reconcile and Adaptive Planning remain separate

### Open Questions

Keep:

- final R2 stale threshold
- final R4 observation and absence thresholds
- final R5 Project thresholds
- whether all Drop actions require item-level confirmation
- whether exact recommendation count limits belong in product or UX specs
- exact Task split contract
- whether evidence-quality labels are user-visible

Remove:

- whether AI may freely infer semantic groups
- whether AI may vaguely recommend Project review
- whether Project completion proves Goal achievement
- whether AI may autonomously apply consequential actions

---

## 20. Affected Formal Documents — Record Only, Do Not Update Yet

After consolidation, accepted decisions should update or create:

- Reconcile deterministic fact specification
- bounded Reconcile rule catalog
- Reconcile recommendation contract
- protected-item policy
- ownership-first grouping specification
- bulk-action safety policy
- Project and Goal escalation guardrails
- consequence-disclosure specification
- Reconcile-to-Adaptive-Planning evidence handoff
- data and event specification in Discussion 019
- runtime and rule-versioning specification in Discussion 020
- validation plan in Discussion 021
- implementation plan in Discussion 022

Potential ADRs:

- bounded rule-driven AI instead of free semantic interpretation
- ownership-first Reconcile grouping
- no autonomous consequential Reconcile actions
- explicit protected-item safety boundary
- separate Reconcile resolution from Adaptive Planning generation

No Mind Map or formal document is updated before consolidation.

---

## 21. Structured Review Request for Claude

Review this exact revision for coherence, safety, MVP simplicity, and closure readiness.

For every issue, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected decision or rule
4. concrete failure scenario
5. why it conflicts or is incomplete
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- contradictions with Discussions 012, 015, or 016
- correction of the original direct-Goal ownership error
- whether any AI responsibility still permits guessing
- whether every recommendation is tied to an accepted rule
- whether the initial R1–R6 catalog is appropriately small
- R2 stale threshold edge cases
- R4 observed-occurrence and absence-contamination requirements
- R5 Project threshold edge cases
- protected-item completeness
- whether bulk Drop is safe enough for MVP
- whether Project target-date extension belongs inside Reconcile
- whether Task Split is underspecified
- whether recommendation frequency limits are coherent
- whether Goal and Project escalation remain sufficiently bounded
- whether standalone work remains fully supported
- whether Reconcile remains separate from Adaptive Planning
- whether any rule cannot be tested deterministically

Do not expand into database schemas, API payloads, provider choice, prompt design, UI styling, analytics implementation, full Adaptive Planning thresholds, or implementation sequencing.

This discussion remains open until Claude reviews this exact revision, Mahdi resolves accepted findings, and final Review Resolution, Mind Map Impact, and Affected Formal Documents are recorded.
