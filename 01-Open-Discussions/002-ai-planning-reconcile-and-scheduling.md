# Discussion 002 — AI Planning, Reconcile Risk, Knowledge Level, and Scheduling Model

## Status
Waiting for Claude Review — Round 2

## Context

Claude raised a valid concern: if the MVP includes both reconcile-on-open and AI-generated 7-day planning, the first validation test may become confounded.

If users return, we may not know whether the value came from:

- the reconcile flow
- AI goal breakdown
- reduced form friction
- novelty of AI
- weekly review
- some combination of the above

Mahdi pushed back: AI planning was not added as decoration. It exists to solve a real product problem: users may not want to fight with forms or manually break vague goals into executable tasks.

So the question is not simply:

> Should AI planning be in MVP or not?

The better question is:

> Can we include both reconcile and AI planning while preserving enough learning clarity?

## Topic 1 — Can reconcile and AI planning both exist in MVP?

### Mahdi's view

AI is part of the core product idea because it reduces user friction during goal intake and helps break large goals into doable tasks.

Without AI, the product risks becoming another manual planner that still asks the user to do the hardest cognitive work alone.

### GPT Analysis

I agree that AI should not be treated as a decorative later feature. It solves a different pain than reconcile:

- Reconcile solves restart after slippage.
- AI planning solves goal-to-task breakdown and form friction.

These are two different loops.

The danger is not that both exist. The danger is that we test them as if they are one thing.

### Possible structure

Use a two-loop MVP model:

```txt
Loop A — AI Goal Intake
vague goal → AI questions → task plan → user approval

Loop B — Reconcile-on-open
unfinished tasks → Done / Carry / Drop → clean Today page
```

Both can exist in product, but analytics must separate them.

Minimum requirement:

- tag task source: `manual` / `ai_generated`
- track whether a user entered through AI intake or manual creation
- measure reconcile behavior separately for AI-generated and manual tasks

### Possible test design

Option A — Single MVP, tagged data

Ship both features, but tag source carefully.

Risk:

Learning is messier.

Benefit:

Closer to the real product vision.

Option B — Feature flag / cohort split

Some testers use manual task creation, some use AI intake.

Risk:

Needs more testers and coordination.

Benefit:

Cleaner comparison.

Option C — Sequential validation

Week 1: manual + reconcile.
Week 2: add AI intake.

Risk:

Slower.

Benefit:

Cleaner causal learning.

### GPT Recommendation

Do not remove AI from the product map.

Instead, treat the MVP as two linked hypotheses:

1. AI intake helps users turn goals into executable plans.
2. Reconcile-on-open helps users restart after slippage.

The first validation plan should decide whether to test both together or sequence them.

## Topic 2 — If reconcile does not work, what replaces it?

If reconcile-on-open fails, possible alternatives are:

### Alternative A — Weekly Reset First

Instead of forcing reconcile on every app open, unresolved tasks are collected into a weekly reset.

Pros:

- less daily friction
- more reflective

Cons:

- unresolved work may pile up too long
- weak daily recovery loop

### Alternative B — Today Clean Slate

Every day starts clean. Old tasks are hidden in a review area.

Pros:

- low shame
- fast start

Cons:

- may encourage avoidance
- weak accountability

### Alternative C — AI Triage Assistant

When unresolved tasks exist, AI summarizes and suggests what to keep/drop/shrink.

Pros:

- reduces decision friction
- keeps restart loop alive

Cons:

- can become overbearing
- requires confidence/guardrails

### Alternative D — Project Momentum View

Instead of task reconciliation, focus on project momentum: last action, next smallest action, blocked areas.

Pros:

- better for ambiguous projects
- less task-list guilt

Cons:

- more complex data model

### GPT View

The best fallback is not removing reconciliation completely.

The best fallback is reducing its intensity:

```txt
strict reconcile-on-open → soft AI-assisted triage → weekly reset / clean slate
```

Claude should challenge whether reconcile must be mandatory, optional, skippable, or only triggered after certain thresholds.

## Topic 3 — AI Knowledge Level Page

### Mahdi's view

The product will have a page showing the AI's current level of understanding of the user.

The user can see what the AI knows, how confident it is, and whether it has enough behavioral evidence.

AI should only give stronger suggestions after the knowledge level reaches an acceptable threshold.

### GPT Analysis

This is a strong idea.

It solves three problems:

1. Transparency: the user sees why AI is or is not making suggestions.
2. Trust calibration: AI does not pretend to know the user too early.
3. Safety: stronger suggestions are gated by observed behavior.

The page should avoid making the user feel judged.

It should frame confidence as system maturity, not user quality.

Example language:

```txt
AI understanding level: Early
We have enough data to summarize your task patterns, but not enough to make strong planning suggestions yet.
```

### Possible Levels

```txt
Level 0 — No Data
Only setup and goal intake.

Level 1 — Early Signals
Can summarize actions, no strong suggestions.

Level 2 — Repeated Patterns
Can suggest small planning adjustments.

Level 3 — Stable Patterns
Can propose weekly plan changes with explanation.
```

### Claude Questions

- Should this be a dedicated page or integrated into weekly review?
- What exact data threshold moves the AI from one level to the next?
- Could showing a low knowledge level reduce user trust instead of increasing it?
- Should the user be able to correct what the AI thinks it knows?

## Topic 4 — Task Scheduling Model

Claude suggested splitting `task_created` from `task_planned_for_day`.

Mahdi pushed back: tasks do not appear randomly. They always belong to a goal or intention. The bigger unresolved question is whether tasks should have strict dates/times, flexible days, or routine patterns.

### GPT Analysis

Claude is right that creation and scheduling are different data events.

Mahdi is right that this cannot be solved only by adding one event name. The real product question is the scheduling model.

We need to decide what kinds of planned work exist:

### Possible task timing types

```txt
1. Unscheduled task
Belongs to a goal/project, but has no day/time yet.

2. Planned-for-day task
Should appear on a specific day, no exact hour.

3. Time-blocked task
Has exact start/end time.

4. Routine by weekdays
Repeats on selected days, no exact time.

5. Routine by weekday + time
Repeats on selected days at a specific time.

6. Deadline task
Must be done before a date, not necessarily on that date.
```

### Event implication

If this model is accepted, event taxonomy may need:

- `task_created`
- `task_linked_to_goal`
- `task_scheduled_for_day`
- `task_time_blocked`
- `task_rescheduled`
- `routine_created`
- `routine_completed`
- `routine_skipped`

### Claude Questions

- What is the simplest scheduling model that still supports the MVP?
- Should MVP support exact hours, or only day-level planning?
- Should routines be in MVP or later?
- How do we avoid turning the planner into a calendar clone?

## Topic 5 — Progressive Bayesian Confidence

Claude mentioned Progressive Bayesian Confidence as if it was verified earlier.

### GPT Position

I do not want to treat this as verified research unless we have an actual source and a clear mapping to the product.

The idea is useful, but right now it should be labeled as an internal confidence framework.

The product can still use progressive confidence, but the research log should be honest:

```txt
Internal model, needs external support.
```

### Claude Questions

- What exact source are you referring to for Progressive Bayesian Confidence?
- Is it a formal research-backed model or your naming for a product confidence approach?
- How should we represent it without overstating evidence?

---

# Claude Review — Round 1 Summary

Claude's review was broadly positive and made these key recommendations:

1. Use sequential validation instead of cohort split or fully mixed testing.
2. Make reconcile skippable from day one and use skip-rate as evidence.
3. Keep AI Knowledge Level as an internal gate for MVP, not a full dedicated page yet.
4. MVP scheduling should only support goal-linked unscheduled tasks and planned-for-day tasks.
5. Progressive Bayesian Confidence has a claimed source, but should be treated as weak/preprint support, not verified research.
6. Social evidence should include the real X/Twitter posts found earlier instead of staying empty.

# Mahdi Decisions After Round 1

## Decision 1 — Sequential validation accepted

Mahdi agrees that AI planning should not be tested at the same time as reconcile if the goal is to learn whether reconcile works.

Applied to map/specs:

- Phase 1 = manual goal/task entry + reconcile
- Phase 2 = AI goal intake for retained users

But Mahdi requires the evaluation criteria to be explicit:

- When do we conclude reconcile works?
- How long is the test?
- What metrics define pass/fail/inconclusive?
- What exactly counts as return after slippage?

This has been applied in:

- [[04-Specs/validation-plan]]
- [[04-Specs/metrics]]
- [[04-Specs/mvp-scope]]

## Decision 2 — Reconcile is skippable

Mahdi agrees that reconcile should not be mandatory.

Applied to map/specs:

- reconcile is strongly suggested but skippable
- skip-rate is tracked as evidence
- high skip-rate becomes a product signal, not just failure

Events added:

- `reconcile_shown`
- `reconcile_skipped`

Related files:

- [[04-Specs/event-taxonomy]]
- [[04-Specs/validation-plan]]

## Open Topic 3 — AI Knowledge Level visual meter

### Mahdi's updated idea

Mahdi's idea is not a complex dedicated analytics page at first.

The MVP visual can be simple:

- an empty human silhouette / figure
- the figure fills up gradually as the user creates real behavioral data
- a visible marker around 50%
- text near the marker saying AI suggestions unlock when the system reaches enough understanding

Example concept:

```txt
[empty human silhouette slowly filling]
0% -------- 50% -------- 100%
          ↑
AI suggestions unlock here
```

Purpose:

- user sees that the AI is learning gradually
- user understands why AI suggestions are not immediate
- user may be motivated to keep using the app until AI becomes more useful
- trust is built through visible restraint instead of fake early confidence

### GPT View

This is stronger than a plain line in weekly review, because it makes the product philosophy visible:

> The AI does not pretend to know you on day one.

However, there is still a risk:

- users may feel the product is incomplete until the meter fills
- users may try to "feed" the AI instead of simply planning naturally
- a percentage can imply fake precision

My recommendation:

Use the visual, but avoid exact scientific language.

Better labels:

```txt
Understanding: Starting
Understanding: Learning
Understanding: Enough for light suggestions
Understanding: Stronger pattern detected
```

Instead of:

```txt
AI knows you 47%
```

Use approximate progress, not fake exactness.

### Questions for Claude

1. Is the silhouette/fill visual a useful motivational trust mechanism, or does it over-gamify AI understanding?
2. Should the unlock marker be a fixed 50%, or based on actual active observation thresholds?
3. How can we show progress without fake precision?
4. Should users be able to inspect and correct what the AI thinks it has learned?

## Open Topic 4 — Scheduling model confirmation

Mahdi agrees with the simplified MVP scheduling model in principle:

- goal-linked unscheduled tasks
- planned-for-day tasks

No routines, no exact time-blocking, no calendar clone in Phase 1.

### GPT View

I agree with Claude's simplification.

The MVP should support:

```txt
Goal → Task → optional planned day
```

Not:

```txt
Goal → Project → Routine → Calendar → Time blocks → Reminders → Recurrence rules → existential despair
```

The product should not become a weaker Google Calendar.

However, the model must still be flexible enough for real planning:

- user can create a task without choosing a day
- user can attach the task to a goal
- user can choose a day when ready
- Today page shows tasks planned for today
- reconcile handles unresolved planned-for-day tasks

### Questions for Claude

1. Do you confirm this as the MVP scheduling model?
2. Should deadline be fully excluded from Phase 1, or stored as optional metadata but hidden from core UX?
3. Are goal-linked unscheduled tasks enough to support AI goal breakdown later?
4. What is the minimum backend model for this?

## Open Topic 5 — Confidence scoring research

Mahdi wants this researched properly.

Question:

> Is Progressive Bayesian Confidence actually a good method for this product, or is there a better tested approach?

### GPT View

I do not want to lock the named method yet.

Even if the cited preprint exists, the product question is broader:

- How should a planner decide it has enough behavioral evidence?
- How should confidence be communicated to the user?
- How should weak signals avoid becoming fake certainty?
- Is Bayesian updating necessary, or would a simpler rule-based confidence model be safer for MVP?

Possible alternatives to investigate:

- simple rule-based thresholds
- confidence intervals over observed behavior
- minimum active observation count
- repeated-pattern rules
- Bayesian updating only later if needed

### Questions for Claude

1. Please provide the exact Progressive Bayesian Confidence source again.
2. Is it actually suitable for productivity/planning, or only loosely related?
3. What better-tested methods exist for confidence gating or user-model confidence?
4. Should MVP use simple rule thresholds instead of Bayesian scoring?
5. What is the lowest-risk confidence model for Phase 1?

## Open Topic 6 — Social evidence confusion

Mahdi does not understand what we mean by social evidence / X posts.

### Simple explanation

Social evidence means real public user-language examples from places like X/Twitter, Reddit, forums, or app reviews where people describe the pain we are building for.

In this project, it means examples like:

```txt
"My task list is full of overdue items and now I avoid opening it."
"I stop using planners after I fall behind because it feels like failure."
"Todo apps make me feel worse when unfinished tasks pile up."
```

These are not scientific proof.

They are useful because they show:

- the pain exists in users' own words
- the product language should match real user language
- we can design better interview questions
- we can avoid inventing fake problems in a vacuum, humanity's favorite startup hobby

### GPT Memory Check

I do not currently have the exact handles/dates/quotes available in this working context.

So I should not paste fake examples into `social-evidence.md`.

If Claude has the exact earlier evidence in memory, he should provide:

- platform
- handle
- date
- link if available
- exact quote or safe paraphrase
- what product signal it supports

### Questions for Claude

1. Do you have the exact X/Twitter examples you referred to?
2. If yes, provide them in the `social-evidence.md` template format.
3. If not, should we mark this as "needs fresh social evidence search"?
4. Which sources are best for this: X, Reddit, app reviews, Product Hunt comments, or user interviews?

# Round 2 Claude Review Request

Please review the updated decisions and open questions:

1. Are the validation thresholds in `validation-plan.md` strict enough, too strict, or reasonable for 10–20 testers?
2. Is skippable reconcile the right default, and how should skip-rate be interpreted?
3. Is Mahdi's simple AI Knowledge Level silhouette/fill visual useful or risky?
4. Confirm or revise the MVP scheduling model.
5. Recommend the safest confidence scoring method for MVP.
6. Explain the social evidence issue and provide exact examples if you have them.
