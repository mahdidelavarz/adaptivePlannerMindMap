# Discussion 003 — Phase 1 Implementation Sequence

## Status
Claude Round 1 reviewed — Mahdi decisions applied, one skipped-task question still open

## Context

The product direction is now mostly locked for Phase 1:

```txt
Manual goal/task entry
Reconcile-on-open
Skippable reconcile
Goal → Task → optional planned day
AI implemented in stack, but hidden/disabled for user-facing Phase 1 validation
No visible AI Knowledge meter in Phase 1
No routines / time-blocking / deadlines in Phase 1
Rule-based confidence only
14-day validation test
```

Important clarification from Mahdi:

AI should not be removed from architecture or implementation planning.

The AI layer can be fully implemented and considered in the stack/map, but for Phase 1 validation it should be disabled or hidden from users so reconcile can be tested cleanly.

The next question is not "what is the product?"

The next question is:

> What should we define next so the designer and backend developer can start without inventing incompatible versions of the same product?

GPT proposes the next three implementation specs:

1. Day-0 Onboarding Spec
2. Reconcile UX Spec
3. Phase 1 Data Model Spec

This discussion is for Claude to critique the sequence, reduce scope, and identify missing constraints before these become real spec files.

---

# Claude Round 1 Summary

Claude agreed with the sequence:

```txt
Day-0 Onboarding
→ Reconcile UX
→ Phase 1 Data Model
```

But identified several corrections before these become formal specs:

1. Day-0 should not hardcode exact Reconcile action copy.
2. Do not add AI to Day-0 just to make onboarding more exciting.
3. Carry should default to Today, with one-tap change.
4. Drop should not require confirmation; use undo toast.
5. Skipped reconcile should not re-trigger again the same day.
6. DailyRollup should not be a stored Phase 1 table; compute it from event queries.
7. Carried is not a Task status.
8. Event metadata should be minimal and metric-driven.

Mahdi accepted these, with one added open question:

> What should happen to skipped tasks? Should the user have a place to view them or bring them back into plans later?

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
8. Keep AI implementation hidden from user-facing Phase 1 validation.

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
- no routines
- no time-blocking
- no deadlines
- no productivity score
- no shame copy

## AI Architecture Note

AI may exist in the codebase, stack, and architecture during Phase 1.

But for Phase 1 validation:

- do not show AI-generated plans to users
- do not expose AI Knowledge meter
- do not let AI planning affect success metrics
- keep source tagging ready for Phase 2

This preserves the long-term AI vision without contaminating the first Reconcile validation.

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
- keep AI source tagging ready for Phase 2

## Resolved Questions

### Is the Day-0 flow too manual and boring without AI?

Decision:

Do not add AI to Day-0 Phase 1 just to make it feel more exciting.

Reason:

The target segment is developers / solo builders / technical users, who can handle manual task creation. Reintroducing AI here would reopen the same confound Phase 1 is designed to avoid.

If Day-0 completion is low during the test, treat that as a signal later.

## Remaining Questions for Claude

1. Is 3–5 tasks too many for Day-0?
2. Should goal creation be mandatory, or can users create standalone tasks?
3. Should Day-0 force at least one task planned for today?
4. Should the user see a vague preview of reconcile during onboarding, or only learn it later when needed?
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

## Re-trigger rule after skip

If the user skips reconcile, do not show reconcile again later that same day.

Reason:

Repeated same-day prompting becomes nagging and pollutes the skip-rate signal.

Reconcile may reappear on the next app open on a later day if unresolved planned-for-day tasks still exist.

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
- no blocking dead-end
- no productivity score

## Suggested Completion State

After reconcile:

```txt
Today is clean.
Here is what you chose to keep.
```

Then route to Today page.

## Open Question — What happens to skipped tasks?

Mahdi added an important unresolved question:

If the user skips reconcile, what should happen to those unresolved tasks?

Possible models:

### Option A — Hidden until next app open on later day

Skipped tasks remain unresolved but do not interrupt the user again that day.

Pros:

- low friction
- protects skip signal

Cons:

- tasks may feel hidden or lost

### Option B — Today banner / backlog access

Skipped tasks are accessible from a calm Today banner:

```txt
3 unfinished tasks from earlier
[Review]
```

Pros:

- user can return voluntarily
- keeps tasks discoverable

Cons:

- still adds visual weight to Today

### Option C — Separate "Unresolved" or "Later" page

Skipped/unresolved tasks live in a separate page where users can review and bring them back into plans.

Pros:

- clean Today page
- user has a recovery place

Cons:

- adds navigation and product complexity
- may become a guilt pile with better branding, humanity's favorite UX costume change

### GPT Leaning

For Phase 1, prefer Option B:

```txt
Calm Today banner + manual Review action
```

Do not force the reconcile flow again the same day.

Do not create a full separate unresolved-task page unless testing shows users feel tasks are lost.

## Questions for Claude

1. After skip, should unresolved tasks be hidden, shown as a Today banner, or moved to a separate page?
2. How do we keep skipped tasks discoverable without creating a guilt pile?
3. Should user be able to bring skipped tasks back into today's plan manually?
4. Should skipped tasks count against validation if user later completes them without reconciling?
5. Is a separate unresolved-task page too much for Phase 1?

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

The data model should support Phase 1 validation and Phase 2 AI readiness, not the full long-term Adaptive Life OS.

AI can be implemented in the stack, but user-facing AI planning is hidden during Phase 1 validation.

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

### Task events

Useful metadata:

```txt
previousPlannedForDate
newPlannedForDate
carryCountAtEvent
```

### Reconcile events

Useful metadata:

```txt
unresolvedTaskCountShown
```

Do not add speculative metadata just because it might be useful someday. That is how databases become attics.

## Explicitly excluded from Phase 1 schema

- stored DailyRollup table
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

Should `source` exist in Phase 1 even though user-facing AI planning is hidden?

GPT recommendation:

Yes. Set all Phase 1 user-created tasks to `manual`. This prepares Phase 2 comparison without overbuilding.

## Resolved Cuts Before Formal Specs

Apply these cuts before creating formal specs:

1. Remove hardcoded Done/Carry/Drop copy from Day-0.
2. Do not add AI to user-facing Day-0 Phase 1.
3. Carry defaults to Today, with one-tap date change.
4. Drop uses undo toast, no confirmation.
5. No same-day Reconcile re-trigger after skip.
6. No stored DailyRollup table in Phase 1.
7. Carried is not a Task status.
8. Event metadata stays metric-driven.

## Remaining Questions for Claude

1. Is the skipped-task banner approach the right Phase 1 compromise?
2. Should skipped tasks be accessible from Today or from a separate page?
3. Is this data model still enough if AI is fully implemented but hidden in Phase 1?
4. What minimal AI-related backend fields are needed now, if any, without polluting Phase 1 validation?
5. Should `source: ai_generated` exist before any user-facing AI flow is enabled?

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

Please review the updated decisions:

1. Is it correct to implement AI in the stack but hide it from Phase 1 users?
2. Is the Day-0 copy now decoupled enough from Reconcile UX?
3. Is the Reconcile UX decision set correct: Carry default Today, Drop undo toast, no same-day skip re-trigger?
4. What should happen to skipped/unresolved tasks after user skips reconcile?
5. Is removing stored DailyRollup correct?
6. Is "carried is not status" enough for backend clarity?
7. Is the minimal event metadata sufficient for validation?
8. What should be cut or added before formal specs are created?
