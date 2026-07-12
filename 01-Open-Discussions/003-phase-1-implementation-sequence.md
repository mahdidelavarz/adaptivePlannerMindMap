# Discussion 003 — Phase 1 Implementation Sequence

## Status
Claude reviewed — Mahdi decisions applied. Ready to turn into formal specs.

## Context

The product direction is now mostly locked for Phase 1:

```txt
Manual goal/task entry
Reconcile-on-open
Skippable reconcile
Goal → Task → optional planned day
AI-ready schema and architecture only in Phase 1
No actual LLM/AI planning implementation in Phase 1
No visible AI Knowledge meter in Phase 1
No routines / time-blocking / deadlines in Phase 1
Rule-based confidence only
14-day validation test
```

The next question is not "what is the product?"

The next question is:

> What should we define next so the designer and backend developer can start without inventing incompatible versions of the same product?

GPT proposes the next three implementation specs:

1. Day-0 Onboarding Spec
2. Reconcile UX Spec
3. Phase 1 Data Model Spec

This discussion collected Claude's review, Mahdi's decisions, and the final constraints before these become real spec files.

---

# Claude Review Summary

Claude agreed with the sequence:

```txt
Day-0 Onboarding
→ Reconcile UX
→ Phase 1 Data Model
```

Claude identified these corrections:

1. Day-0 should not hardcode exact Reconcile action copy.
2. Do not add AI to Day-0 just to make onboarding more exciting.
3. Carry should default to Today, with one-tap change.
4. Drop should not require confirmation; use undo toast.
5. Skipped reconcile should not re-trigger again the same day.
6. DailyRollup should not be a stored Phase 1 table; compute it from event queries.
7. Carried is not a Task status.
8. Event metadata should be minimal and metric-driven.
9. AI Phase 1 should mean schema/architecture readiness only, not actual LLM implementation.
10. After skipped reconcile, use Option A for Phase 1: no banner, no same-day resurfacing, unresolved tasks reappear only on a later app open.

Mahdi accepted these decisions.

---

# Important Clarification — AI in Phase 1

## Decision

Phase 1 should be **AI-ready**, not **AI-implemented**.

This means:

```txt
Keep:
- source field
- ai_generated enum value
- AI docs / guardrails
- Phase 2 AI planning in the map
- architecture awareness that AI will exist later

Defer:
- actual LLM integration
- prompt execution
- AI plan generation endpoint
- structured output enforcement runtime
- hidden AI feature flag
- AI API calls
- AI Knowledge meter UI
```

## Reason

Schema readiness is cheap and useful.

Actual AI implementation is not cheap. It adds:

- engineering time
- API cost risk
- hidden code path bugs
- feature flag leakage risk
- extra QA surface
- possible contamination of Phase 1 validation

Phase 1 exists to validate Reconcile. The AI vision stays in the map, but real AI execution starts in Phase 2.

## Final phrasing

Use this:

```txt
Phase 1 keeps the product AI-ready at the schema and architecture level, but does not implement user-facing or hidden LLM functionality.
```

Do not use this:

```txt
AI is fully implemented but hidden.
```

That sentence is too ambiguous and can quietly smuggle scope creep into the build. Truly majestic how one sentence can bankrupt a sprint.

---

# Step 1 — Day-0 Onboarding Spec

## Recommended file

```txt
04-Specs/day-0-onboarding.md
```

## Goal of Day-0

Get the user from zero to a real Today page with real work in under 3–5 minutes.

## Day-0 should accomplish

1. Explain the product promise briefly.
2. Ask the user for one real goal/project.
3. Help the user create 3–5 real tasks manually.
4. Let the user choose which tasks are planned for today or tomorrow.
5. Show the Today page.
6. Avoid AI personalization claims.
7. Explain that the system learns from real behavior over time.
8. Keep AI hidden from user-facing Phase 1 validation.

## Proposed Day-0 Flow

```txt
1. Welcome / Promise
   "This planner helps you recover when plans break, not punish you for falling behind."

2. Create first goal
   User enters one real goal/project.

3. Create first tasks
   User adds 3–5 tasks related to that goal.

4. Choose planned day
   User selects Today / Tomorrow / Later for each task.

5. Today page
   User sees only tasks planned for today.

6. Micro-explanation
   "If something is unfinished later, we'll help you sort it out."
```

## Applied Claude Correction

Day-0 must not hardcode the exact Reconcile action vocabulary.

Do not say:

```txt
we will help you decide: done, carry, or drop
```

Use softer copy instead:

```txt
If something is unfinished later, we'll help you sort it out.
```

Reason:

Reconcile UX should own the exact action language, tone, and behavior.

## What Day-0 must not do

- no personality quiz
- no long questionnaire
- no visible AI Knowledge meter
- no user-facing AI-generated task plan in Phase 1
- no hidden LLM execution
- no routines
- no time-blocking
- no deadlines
- no productivity score
- no shame copy

## First Success Moment

The first success moment is not account creation.

It is:

> The user sees a clean Today page with a few real tasks they intentionally planned.

## Designer Notes

Designer should focus on:

- low-friction task creation
- clear empty states
- calm copy
- fast path to Today page
- avoiding setup fatigue

## Backend Notes

Backend must support:

- create goal
- create task
- link task to goal
- schedule task for day
- mark source as `manual`
- log onboarding events
- keep source tagging ready for Phase 2 AI comparison

## Resolved Decision

Do not add AI to user-facing Day-0 Phase 1 just to make it feel more exciting.

Reason:

The target segment is developers / solo builders / technical users, who can handle manual task creation. Reintroducing AI here would reopen the same confound Phase 1 is designed to avoid.

If Day-0 completion is low during the test, treat that as a signal later.

## Remaining Day-0 Questions

1. Is 3–5 tasks too many for Day-0?
2. Should goal creation be mandatory, or can users create standalone tasks?
3. Should Day-0 force at least one task planned for today?
4. Should the user see a vague preview of reconcile during onboarding, or only learn it later when needed?
5. Is 3–5 minutes realistic?

---

# Step 2 — Reconcile UX Spec

## Recommended file

```txt
04-Specs/reconcile-ux.md
```

## Goal of Reconcile UX

Help the user resolve unfinished planned work without shame and get back to Today.

## Trigger

Show reconcile when:

- user opens the app, and
- there are unresolved planned-for-day tasks from a previous day.

## Reconcile should be skippable

The user can choose:

```txt
Handle now
Skip for now
```

Skip must log:

```txt
reconcile_skipped
```

If the user skips and still takes meaningful action within 5 minutes, log:

```txt
meaningful_action_after_reconcile_skip
```

## Re-trigger rule after skipped reconcile

If the user skips reconcile, do not show reconcile again later that same day.

Reason:

Repeated same-day prompting becomes nagging and pollutes the skip-rate signal.

Reconcile may reappear on the next app open on a later day if unresolved planned-for-day tasks still exist.

## After skipped reconcile — Phase 1 decision

Use **Option A** for Phase 1.

```txt
Skipped reconcile
→ no persistent Today banner
→ no separate unresolved page
→ no same-day re-trigger
→ unresolved tasks remain unresolved
→ reconcile can appear again on next app open on a later day
```

## Why Option A

Option A has the lowest build cost and cleanest validation signal.

Avoid in Phase 1:

- Today banner for unresolved tasks
- separate unresolved/backlog page
- manual review link after skip

Reason:

A persistent banner or separate page creates a second path back to unresolved tasks, which contaminates the skip signal.

If the user skips reconcile and still acts, we want to know whether that action was organic, not prompted by a second UI surface.

If testers complain that skipped reconcile made old tasks feel lost, then a calm review banner or separate recovery page can be added later with evidence behind it.

## Proposed UX Model

Use a lightweight full-page or focused panel, not an aggressive modal.

Copy should frame it as cleanup, not judgment.

Example:

```txt
You have 3 unfinished tasks from earlier.
Let's clean things up so today stays clear.
```

For each task:

```txt
Task title
[Done] [Carry] [Drop]
```

Optional later action:

```txt
[Skip for now]
```

## Action meanings

### Done

The task was completed outside the app.

Event:

```txt
task_completed
```

### Carry

Move task to a new planned day.

Default:

```txt
Today
```

User can change with one tap:

```txt
Today / Tomorrow / Later
```

Do not force a date choice for every carried task.

Event:

```txt
task_carried
```

### Drop

The task no longer matters.

Do not ask for a reason in Phase 1.

Do not show a confirmation dialog.

Instead, use an undo toast:

```txt
Dropped. Undo
```

Event:

```txt
task_dropped
```

### Skip for now

User avoids reconcile temporarily.

Event:

```txt
reconcile_skipped
```

## What Reconcile must not do

- no red overdue labels
- no failure language
- no guilt copy
- no forced reason for dropping
- no drop confirmation dialog
- no same-day re-trigger after skip
- no persistent unresolved-task banner in Phase 1
- no separate unresolved-task page in Phase 1
- no blocking dead-end
- no productivity score

## Suggested Completion State

After reconcile:

```txt
Today is clean.
Here is what you chose to keep.
```

Then route to Today page.

---

# Step 3 — Phase 1 Data Model Spec

## Recommended file

```txt
04-Specs/data-model-phase-1.md
```

## Core Principle

The data model should support Phase 1 validation and Phase 2 AI readiness, not the full long-term Adaptive Life OS.

Phase 1 should keep the schema ready for AI-generated tasks later, but must not implement runtime AI behavior.

## Minimum Entities

### Goal

```ts
type Goal = {
  id: string;
  userId: string;
  title: string;
  createdAt: string;
  archivedAt?: string | null;
};
```

### Task

```ts
type Task = {
  id: string;
  userId: string;
  goalId?: string | null;
  title: string;
  status: "active" | "completed" | "dropped";
  source: "manual" | "ai_generated" | "system";
  plannedForDate?: string | null;
  carryCount: number;
  createdAt: string;
  completedAt?: string | null;
  droppedAt?: string | null;
};
```

## Carried is not a status

A carried task remains:

```txt
status = active
```

Carry changes:

```txt
plannedForDate
carryCount
```

and logs:

```txt
task_carried
```

Do not add `carried` as a fourth task status.

### Event

```ts
type Event = {
  id: string;
  userId: string;
  type: string;
  entityType: "goal" | "task" | "plan" | "review" | "app";
  entityId?: string | null;
  source: "manual" | "ai_generated" | "system";
  timestamp: string;
  metadata?: Record<string, unknown>;
};
```

## No stored DailyRollup table in Phase 1

Do not create a stored `DailyRollup` table in Phase 1.

Reason:

For 10–20 testers over 14 days, event volume is small enough to compute rollups through queries.

A stored rollup table creates sync drift risk:

```txt
event log says one thing
rollup table says another
```

Use query-based rollups first. Materialize later only if volume or performance requires it.

## Minimal Event Metadata

Only store metadata required by defined metrics.

### `task_carried`

```txt
previousPlannedForDate
newPlannedForDate
carryCountAtEvent
```

### `task_dropped`

```txt
previousPlannedForDate
carryCountAtEvent
```

Reason:

Capturing `carryCountAtEvent` on drop helps later identify patterns like:

```txt
task carried 3 times, then dropped
```

This may suggest the task was too vague, too large, or no longer relevant.

### `reconcile_shown`

```txt
unresolvedTaskCountShown
```

### `reconcile_skipped`

```txt
unresolvedTaskCountShown
```

Do not add speculative metadata just because it might be useful someday. That is how databases become attics.

## Explicitly excluded from Phase 1 schema

- stored DailyRollup table
- runtime AI integration
- AI user profile table
- personality model
- routines
- time blocks
- deadline fields
- reminders
- recurring rules
- complex project hierarchy
- calendar integration

## Important Data Decisions

### carryCount

Store it on Task for simple UI/performance, but treat Event as source of truth.

### plannedForDate

A task can exist without a planned date.

Tasks can be goal-linked but unscheduled.

### dropped vs archived

Use `dropped` as status in Phase 1.

Archive can be later.

### source

Keep `source` in Phase 1 even though user-facing AI planning is hidden.

Set all Phase 1 user-created tasks to:

```txt
source = manual
```

This prepares Phase 2 comparison without overbuilding.

---

# Resolved Cuts Before Formal Specs

Apply these cuts before creating formal specs:

1. Remove hardcoded Done/Carry/Drop copy from Day-0.
2. Do not add user-facing AI to Day-0 Phase 1.
3. Do not implement runtime AI/LLM functionality in Phase 1.
4. Carry defaults to Today, with one-tap date change.
5. Drop uses undo toast, no confirmation.
6. No same-day Reconcile re-trigger after skip.
7. After skipped reconcile, use Option A: no banner, no separate page, reappear later.
8. No stored DailyRollup table in Phase 1.
9. Carried is not a Task status.
10. Event metadata stays metric-driven.
11. Capture `carryCountAtEvent` on `task_dropped` as well as `task_carried`.

---

# Remaining Open Questions

These are still open before formal specs:

## Day-0 Onboarding

1. Is 3–5 tasks too many for Day-0?
2. Should goal creation be mandatory, or can users create standalone tasks?
3. Should Day-0 force at least one task planned for today?
4. Should the user see a vague preview of reconcile during onboarding, or only learn it later when needed?
5. Is 3–5 minutes realistic?

## Reconcile UX

1. Should reconcile appear before Today, or as a focused panel within Today?
2. Is full-page too heavy for Phase 1?
3. Should skipped reconcile reappear on the next calendar day, or the next app open after today?
4. Should `Skip for now` copy be softer, e.g. `Not now`?

## Data Model

1. Is `Goal -> Task -> optional planned day` enough for Phase 1?
2. Should `source: ai_generated` exist before any user-facing AI flow is enabled?
3. What exact metadata should be required for each event type?
4. Should event type be a strict enum from day one?

---

# GPT Suggested Sequence

```txt
1. Create Day-0 Onboarding Spec
2. Create Reconcile UX Spec
3. Create Data Model Phase 1 Spec
4. Review with Claude/backend/designer
5. Mahdi approves
6. Start design + backend implementation planning
```

## Why this order

Day-0 defines the first user experience.

Reconcile defines the core behavior being tested.

Data model defines what must be built to support both.

This order reduces premature backend complexity while keeping implementation grounded.

## Next Recommended Action

Create the three formal specs now, using the resolved decisions above:

```txt
04-Specs/day-0-onboarding.md
04-Specs/reconcile-ux.md
04-Specs/data-model-phase-1.md
```

Then send those files to Claude / backend / designer for review.
