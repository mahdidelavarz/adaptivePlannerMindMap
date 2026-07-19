# Discussion 017A — Goal Continuation and Temporal Checkpoint Resolution

## Status

Open for GPT × Claude review.

This companion amendment records the accepted direction for Goal continuation checks and the unresolved dependency on the temporal-checkpoint amendment in Discussion 012A.

Related discussions:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/012a-temporal-checkpoint-amendment]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/016-reconcile-trigger-and-severity]]
- [[01-Open-Discussions/017-ai-reconcile-intelligence-and-actions]]

---

## 1. Accepted Boundary: No Causal or Emotional Questions

Reconcile AI must not ask:

- why an item was not completed
- why a Routine was missed
- whether the user lost motivation
- when the user expects to feel better
- when the user expects to have more capacity
- what emotional or lifestyle cause explains execution
- any open-ended question intended to derive a metric or rule input

Free-text user input must not feed:

- rule matching
- metric calculation
- inferred rationale
- evidence quality
- recommendation selection

Reconcile intelligence uses only:

```txt
canonical entity state
+ recorded execution events
+ accepted deterministic metrics
+ explicit product actions
```

---

## 2. Goal Continuation Check Is Different

A Goal Continuation Check does not ask for a cause.

It asks for explicit current intent:

> Do you still want to continue this Goal?

This is a deterministic lifecycle checkpoint, not an AI-generated behavioral recommendation.

Allowed options:

```txt
CONTINUE
REVIEW_LATER
ABANDON_GOAL
```

Meaning:

### CONTINUE

```txt
Goal remains ACTIVE
Goal intent is explicitly reaffirmed
next review checkpoint is assigned
child execution remains subject to normal Reconcile rules
```

### REVIEW_LATER

```txt
Goal remains ACTIVE
no lifecycle transition occurs
presentation is deferred to a later Goal review boundary
```

`REVIEW_LATER` does not introduce a `PAUSED` Goal status.

### ABANDON_GOAL

```txt
Goal ACTIVE → ABANDONED
```

The existing child-resolution and lifecycle consequences from Discussion 015 apply.

---

## 3. Neutral Wording Guardrail

The product must not frame the continuation check as accusation, failure, or abandonment detection.

Forbidden framing includes:

```txt
You have not worked on this Goal for a long time.
You have fallen behind. Do you still want this?
It looks like you abandoned this Goal.
```

Required semantic tone:

```txt
neutral
present-focused
non-causal
non-diagnostic
```

Recommended fixed wording:

> Do you still want to continue “{Goal title}”?

Optional factual context may be shown separately only when it is concise and neutral, for example:

```txt
This Goal has execution items that still need review.
```

The context must not imply motive or personal failure.

---

## 4. Trigger Source

The continuation check should not be regenerated directly from absence and stale descendants on every evaluation.

Accepted direction:

```txt
Goal.reviewDate reached
AND Goal.status = ACTIVE
→ Goal Continuation Check becomes eligible
```

Absence and unresolved descendant execution may influence when a future Goal review checkpoint is scheduled, but they do not directly produce a repeated Goal-level question on every app open.

Conceptual separation:

```txt
absence or stale descendant execution
→ may cause or advance a Goal review checkpoint

Goal review checkpoint reached
→ show Goal Continuation Check
```

The exact date model depends on Discussion 012A.

---

## 5. Cadence and Suppression

Goal continuation is heavier than ordinary item-level Reconcile review.

Proposed cadence:

```txt
at most one automatic Goal Continuation Check
per Goal
per 30 local days
```

Guardrails:

- ordinary once-per-day Reconcile presentation rules still apply
- Goal continuation has its own longer per-Goal suppression window
- `REVIEW_LATER` must not cause the question to reappear the next day
- `CONTINUE` assigns or preserves a future Goal review checkpoint
- no duplicate continuation prompt appears within the same evaluated Goal boundary
- the user may still manually open Goal details at any time

The exact 30-day value remains proposed for Claude review.

---

## 6. Relationship to Reconcile Severity

A Goal Continuation Check must not automatically imply `RECOVERY`.

It is a commitment checkpoint, not execution backlog severity.

Proposed rule:

```txt
Goal review due
≠ execution overdue
≠ automatic severity escalation
```

If overdue child execution also exists:

- child execution contributes to normal Reconcile severity under Discussion 016
- the Goal continuation question is grouped as a separate Goal-level checkpoint
- the Goal checkpoint does not multiply child backlog counts

Example:

```txt
Goal review due
+ 1 overdue child Task
→ child backlog may remain LIGHT
→ Goal continuation appears as a separate bounded decision
```

---

## 7. AI Responsibility

AI does not decide whether the Goal still matters.

AI may only:

- render fixed neutral wording
- show deterministic Goal and descendant facts
- explain consequences of the three predefined options
- summarize the explicit decision afterward

AI may not:

- recommend `ABANDON_GOAL`
- infer that the Goal no longer fits the user
- infer that inactivity means low commitment
- rank the three options
- transform free-text explanation into Goal intent
- create a fourth option

---

## 8. Scenario Checks

### Scenario A — Long Absence, No Due Goal Review

```txt
Goal ACTIVE
absence = 30 days
Goal.reviewDate is still in future
```

Expected:

- no Goal Continuation Check yet
- normal child Reconcile may still appear if actionable execution exists
- no Goal abandonment inference

### Scenario B — Goal Review Due, No Execution Backlog

Expected:

- neutral Goal Continuation Check may appear
- no overdue-work framing
- no automatic MEDIUM or RECOVERY severity
- options remain CONTINUE, REVIEW_LATER, ABANDON_GOAL

### Scenario C — Goal Review Due with Overdue Children

Expected:

- child items use normal Reconcile rules
- Goal check is shown separately
- backlog severity comes from actionable child execution
- Goal question does not imply failure

### Scenario D — User Selects CONTINUE

Expected:

- Goal remains ACTIVE
- explicit reaffirmation is recorded
- a future review checkpoint is assigned
- overdue child items remain unresolved until separately acted upon

### Scenario E — User Selects REVIEW_LATER

Expected:

- no lifecycle change
- no next-day repeated prompt
- a later review boundary is assigned or preserved

### Scenario F — User Selects ABANDON_GOAL

Expected:

- user enters the accepted Goal terminal flow
- all child consequences are disclosed
- AI does not choose child actions

---

## 9. Current Proposed Decision

Accept Goal Continuation Check with these guardrails:

```txt
- deterministic, not interpretive
- triggered by an explicit Goal review checkpoint
- fixed neutral wording
- no causal, emotional, motivational, capacity, or predictive questions
- CONTINUE | REVIEW_LATER | ABANDON_GOAL only
- at most once per 30 local days per Goal
- does not independently increase Reconcile severity
- abandonment uses the existing lifecycle resolution flow
```

Final acceptance depends on approval of the temporal-checkpoint model in Discussion 012A.

---

## 10. Open Questions for Claude

### Q1. Is a fixed neutral continuation question still too pressuring when shown beside overdue descendants?

Review whether the Goal checkpoint should appear before, after, or separately from child execution review.

### Q2. Is 30 local days the correct minimum cadence per Goal?

Alternative ranges:

```txt
14 days
30 days
90 days
```

Review the smallest cadence that avoids repeated commitment pressure while remaining useful.

### Q3. Should absence be allowed to advance a system-generated Goal review checkpoint?

Current proposal: yes, but only through deterministic policy and never by directly presenting the question from absence alone.

### Q4. Should `CONTINUE` always assign a new review checkpoint?

Current proposal: yes, to prevent immediate reappearance and preserve temporal visibility.

### Q5. Should `REVIEW_LATER` use a fixed defer interval or require a date-selection action?

Current proposal for low friction: use a deterministic default with optional editing.

### Q6. Should the continuation check remain outside the AI recommendation catalog entirely?

Current proposal: yes. It is a deterministic product action with optional AI wording support.

---

## 11. Mind Map Impact — Record Only, Do Not Apply Yet

### Product Vision

Add:

- Reconcile may confirm current commitment without interrogating causes
- Goal review must remain neutral and non-punitive
- intent is user-confirmed, never inferred from inactivity

### MVP Core Loop

```txt
Goal review checkpoint reached
→ show neutral continuation decision
→ user continues, defers, or starts abandonment flow
→ record explicit intent
→ assign next review checkpoint when Goal stays active
```

### User Flow

```txt
Goal Continuation Check
→ CONTINUE
→ REVIEW_LATER
→ ABANDON_GOAL
```

Child execution review remains separate.

### AI Responsibilities

- render fixed neutral continuation wording
- explain deterministic consequences
- summarize explicit user decision

### AI Guardrails

- no why question
- no emotional or motivational question
- no capacity prediction question
- no abandonment recommendation
- no intent inference from absence
- no free-text consumption for rule matching

### Data Events

Candidates:

```txt
GOAL_CONTINUATION_CHECK_PRESENTED
GOAL_CONTINUATION_CONFIRMED
GOAL_CONTINUATION_DEFERRED
GOAL_ABANDONMENT_FLOW_STARTED
GOAL_REVIEW_CHECKPOINT_ASSIGNED
```

### Traction Metrics

Valid metrics:

- continuation choice distribution
- defer rate
- repeated-prompt rate within suppression window, expected zero
- return to child execution after CONTINUE

Guardrail metrics:

- continuation prompts shown without reached review checkpoint, expected zero
- prompts repeated within 30 days, expected zero
- AI-generated abandonment recommendation rate, expected zero

---

## 12. Affected Formal Documents

If accepted, update:

- Goal lifecycle and intent-confirmation specification
- Reconcile Goal checkpoint specification
- temporal checkpoint amendment in Discussion 012A
- Goal field and default contract in Discussion 014
- Goal review versus execution semantics in Discussion 015
- Reconcile eligibility and severity separation in Discussion 016
- persistence and events in Discussion 019
- deterministic evaluation in Discussion 020
- validation plan in Discussion 021

---

## 13. Closure Condition

This amendment remains open until:

1. Claude reviews the wording, cadence, and trigger source.
2. Discussion 012A resolves the temporal-checkpoint model.
3. Mahdi approves the final Goal continuation rules.
4. Discussion 017 incorporates the accepted amendment and closes.
