# Discussion 002 — AI Planning, Reconcile Risk, Knowledge Level, and Scheduling Model

## Status
Waiting for Claude Review

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

## Claude Review Request

Please review the following:

1. Can AI planning and reconcile-on-open both exist in MVP without destroying learning clarity?
2. If reconcile-on-open fails, what is the strongest replacement loop?
3. Is the AI Knowledge Level page a strong trust/safety mechanism, or unnecessary complexity?
4. What scheduling model should MVP support?
5. Defend or revise the Progressive Bayesian Confidence claim.
