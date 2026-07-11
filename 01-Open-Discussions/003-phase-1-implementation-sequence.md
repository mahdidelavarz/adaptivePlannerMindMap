# Discussion 003 — Phase 1 Implementation Sequence

## Status
Waiting for Claude Review

## Context

The product direction is now mostly locked for Phase 1:

```txt
Manual goal/task entry
Reconcile-on-open
Skippable reconcile
Goal → Task → optional planned day
No AI planning in Phase 1
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

This discussion is for Claude to critique the sequence, reduce scope, and identify missing constraints before these become real spec files.

---

# Step 1 — Day-0 Onboarding Spec

## Why this should come first

Day-0 defines the user's first contact with the product.

If this is vague, every other flow becomes unstable. The Today page, task creation, goal model, and validation test all depend on what the user does in the first few minutes.

A bad onboarding flow can kill the test before reconcile even gets a chance to appear. Humanity has a long tradition of blaming product hypotheses after ruining first-run UX, so let's not contribute to the pile.

## GPT Recommendation

Create:

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
   "If tasks are unfinished later, we will help you decide: done, carry, or drop."
```

## What Day-0 must not do

- no personality quiz
- no long questionnaire
- no AI Knowledge meter
- no AI-generated task plan in Phase 1
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

## Open Questions for Claude

1. Is 3–5 tasks too many for Day-0?
2. Should goal creation be mandatory, or can users create standalone tasks?
3. Should Day-0 force at least one task planned for today?
4. Should the user see a preview of reconcile during onboarding, or only learn it later when needed?
5. Is 3–5 minutes realistic?

---

# Step 2 — Reconcile UX Spec

## Why this comes second

Reconcile is the core Phase 1 hypothesis.

The validation plan depends on whether users complete or skip reconcile after unresolved tasks appear. That means the UX must be defined before build, not improvised by whoever is closest to the keyboard. A tragic governance model, yet popular.

## GPT Recommendation

Create:

```txt
04-Specs/reconcile-ux.md
```

## Goal of Reconcile UX

Help the user resolve yesterday's unfinished work without shame and get back to Today.

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

## Proposed UX Model

Use a lightweight full-page or focused panel, not an aggressive modal.

Copy should frame it as cleanup, not judgment.

Example:

```txt
You have 3 unfinished tasks from earlier.
Let's clean them up so today stays clear.
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

But user can choose:

```txt
Today / Tomorrow / Later
```

Event:

```txt
task_carried
```

### Drop

The task no longer matters.

Do not ask for a reason in Phase 1.

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
- no blocking dead-end
- no productivity score

## Suggested Completion State

After reconcile:

```txt
Today is clean.
Here is what you chose to keep.
```

Then route to Today page.

## Open Questions for Claude

1. Should reconcile appear before Today, or as a banner on Today?
2. Is full-page too heavy for Phase 1?
3. Should Carry default to Today or force a choice?
4. Should Drop require confirmation?
5. Should skipped reconcile reappear later the same day or only next app open?

---

# Step 3 — Phase 1 Data Model Spec

## Why this comes third

After Day-0 and Reconcile UX are defined, the backend model should lock the minimum data needed to support the Phase 1 test.

If data model comes too early, it may overbuild. If it comes too late, frontend and backend improvise different concepts of task, goal, and event. Naturally, computers then obey both versions badly.

## GPT Recommendation

Create:

```txt
04-Specs/data-model-phase-1.md
```

## Core Principle

The data model should support Phase 1 validation, not the full long-term Adaptive Life OS.

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

### DailyRollup

```ts
type DailyRollup = {
  id: string;
  userId: string;
  date: string;
  tasksCreated: number;
  tasksCompleted: number;
  tasksCarried: number;
  tasksDropped: number;
  reconcileShown: number;
  reconcileCompleted: number;
  reconcileSkipped: number;
  firstMeaningfulActionLatencySeconds?: number | null;
};
```

## Explicitly excluded from Phase 1 schema

- routines
- time blocks
- deadline fields
- reminders
- recurring rules
- complex project hierarchy
- AI user profile table
- personality model
- calendar integration

## Important Data Questions

### carryCount

Question:

Should `carryCount` be stored on Task, or derived from events?

GPT recommendation:

Store it on Task for simple UI/performance, but treat Event as source of truth.

### plannedForDate

Question:

Should a task be allowed without a planned date?

GPT recommendation:

Yes. Tasks can be goal-linked but unscheduled.

### dropped vs archived

Question:

Is dropped a status or an archive action?

GPT recommendation:

Use `dropped` as status in Phase 1.

Archive can be later.

### source

Question:

Should `source` exist in Phase 1 even though AI-generated planning is disabled?

GPT recommendation:

Yes. Set all Phase 1 user-created tasks to `manual`. This prepares Phase 2 comparison without overbuilding.

## Open Questions for Claude

1. Is this data model too large or too small for Phase 1?
2. Should `carryCount` be stored or derived only?
3. Should `DailyRollup` be a stored table or generated query?
4. Is `status: active | completed | dropped` enough?
5. Should `plannedForDate` be a direct Task field or separate scheduling table?
6. What event metadata is essential for validation?

---

# GPT Suggested Sequence

```txt
1. Create Day-0 Onboarding Spec
2. Claude reviews
3. Mahdi approves
4. Create Reconcile UX Spec
5. Claude reviews
6. Mahdi approves
7. Create Data Model Phase 1 Spec
8. Claude/backend reviews
9. Mahdi approves
10. Start design + backend implementation planning
```

## Why this order

Day-0 defines the first user experience.

Reconcile defines the core behavior being tested.

Data model defines what must be built to support both.

This order reduces premature backend complexity while keeping implementation grounded.

## Claude Review Request

Please review:

1. Is this the right next sequence?
2. Should Day-0 be specified before Reconcile UX, or should Reconcile UX come first?
3. Is the Day-0 flow too manual and boring without AI?
4. Is the Reconcile UX too heavy or too light?
5. Is the proposed data model enough for Phase 1 validation?
6. What should be cut before these become formal spec files?
