# Discussion 003 — Phase 1 Implementation Sequence

## Status

Mahdi Day-0 answers added — waiting for Claude agreement before Technical Foundation / Stack Decision.

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

The next implementation sequence was originally proposed as:

```txt
Day-0 Onboarding Spec
→ Reconcile UX Spec
→ Phase 1 Data Model Spec
```

Mahdi now raised a better sequencing question:

> Should we harden the stack and technical foundation before writing these formal implementation specs?

GPT view:

Yes. Before turning Day-0, Reconcile UX, and Data Model into formal specs, the project should first lock a short Technical Foundation / Stack Decision.

Suggested sequence:

```txt
1. Technical Foundation / Stack Decision
2. Day-0 Onboarding Spec
3. Reconcile UX Spec
4. Phase 1 Data Model Spec
5. Implementation Plan
```

Reason:

The stack affects auth, event logging, deployment, frontend architecture, backend boundaries, and the eventual AI integration path. Writing UX/data specs before stack decisions can make the docs prettier than the build reality, which is how software projects grow decorative horns and then charge rent.

---

# Claude Review Summary

Claude agreed with the original sequence:

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

That sentence is too ambiguous and can quietly smuggle scope creep into the build.

---

# Step 1 — Day-0 Onboarding Spec

## Recommended file

```txt
04-Specs/day-0-onboarding.md
```

## Goal of Day-0

Get the user from zero to a real Today page with real work quickly, without forcing a rigid setup ritual.

## Mahdi Decisions on Day-0 Open Questions

### Question 1 — Is 3–5 tasks too many for Day-0?

Mahdi decision:

It does not matter. The user can add 1 task, 3 tasks, or even 100 tasks in the first days.

The system should not limit task entry early.

Important distinction:

```txt
User input is free.
System suggestions are gated.
```

The product should allow many tasks, but should only make suggestions after it has enough behavior data.

GPT interpretation:

Do not force a fixed task count in Day-0. Use optional guidance instead:

```txt
Add a few tasks to get started. You can add more later.
```

### Question 2 — Should goal creation be mandatory?

Mahdi decision:

Users can create independent tasks.

A task can be standalone or optionally linked to a goal.

GPT interpretation:

This supports lower-friction Day-0.

Data model should allow:

```txt
goalId = null
```

Goal-linked tasks remain useful, but the app should not force every task into a goal from day one. That would be tidy in a database and annoying in a human life, which is somehow always the tradeoff.

### Question 3 — Should Day-0 force at least one task planned for today?

Mahdi decision:

No force at all.

GPT interpretation:

Do not force the user to plan a task for Today.

However, the UI can still provide a guided default path:

```txt
Want to start today? Pick one task for Today.
```

This keeps the path clear without coercion.

### Question 4 — Should the user see a vague preview of Reconcile during onboarding?

Mahdi asked what this means.

Answer:

A vague preview means one short promise about what happens when plans break, without explaining the exact Reconcile mechanics.

Do not teach:

```txt
Done / Carry / Drop
```

Do not explain the whole flow.

Use only a simple product promise:

```txt
Plans break. When they do, this app helps you clean things up and continue.
```

GPT recommendation:

Include one short vague preview in Day-0, because it explains the product philosophy without turning onboarding into a tutorial graveyard.

Question for Claude:

Is this one-line preview useful, or should Reconcile be discovered only when the user first needs it?

### Question 5 — Is 3–5 minutes realistic?

Mahdi decision:

No fixed opinion yet.

GPT interpretation:

Do not lock a hard time threshold before prototype testing.

Keep a soft target:

```txt
The user should reach a usable Today page quickly.
```

Measure actual onboarding time during prototype/testing instead of pretending we know it in advance, the ancient startup sport.

## Proposed Day-0 Flow

```txt
1. Welcome / Promise
   "This planner helps you recover when plans break, not punish you for falling behind."

2. Optional goal/project
   User may create one goal/project, but this is not mandatory.

3. Task entry
   User can add any number of tasks.
   Suggested copy: "Add a few tasks to get started. You can add more later."

4. Optional planned day
   User may choose Today / Tomorrow / Later for each task.
   No forced planning.

5. Today page
   User sees tasks planned for today if any exist.
   If none exist, Today has a calm empty state and a clear add/plan action.

6. Vague Reconcile preview
   "Plans break. When they do, this app helps you clean things up and continue."
```

## What Day-0 must not do

- no personality quiz
- no long questionnaire
- no visible AI Knowledge meter
- no user-facing AI-generated task plan in Phase 1
- no hidden LLM execution
- no forced goal
- no forced task count
- no forced task planned for Today
- no routines
- no time-blocking
- no deadlines
- no productivity score
- no shame copy

## First Success Moment

The first success moment is not account creation.

It is:

> The user reaches a usable planning surface with tasks they intentionally entered or planned.

If they planned tasks for today, this is the Today page.

If they did not plan anything for today, the success state is still a calm, clear app state where they understand how to add or plan work next.

## Designer Notes

Designer should focus on:

- low-friction task creation
- optional goal linking
- clear empty states
- calm copy
- fast path to Today
- avoiding setup fatigue
- guidance without force

## Backend Notes

Backend must support:

- create optional goal
- create standalone task
- link task to goal optionally
- schedule task for day optionally
- mark source as `manual`
- log onboarding events
- keep source tagging ready for Phase 2 AI comparison

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

Reason:

Option A has the lowest build cost and cleanest validation signal.

A persistent banner or separate page creates a second path back to unresolved tasks, which contaminates the skip signal.

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

For 10–20 testers over 14 days, event volume is small enough to compute rollups through queries.

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

Capturing `carryCountAtEvent` on drop helps later identify patterns like:

```txt
task carried 3 times, then dropped
```

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

---

# Resolved Cuts Before Formal Specs

Apply these cuts before creating formal specs:

1. Remove hardcoded Done/Carry/Drop copy from Day-0.
2. Do not add user-facing AI to Day-0 Phase 1.
3. Do not implement runtime AI/LLM functionality in Phase 1.
4. Allow standalone tasks.
5. Do not force goal creation.
6. Do not force a task count.
7. Do not force planning a task for Today.
8. Use only a vague one-line Reconcile preview, if any.
9. Carry defaults to Today, with one-tap date change.
10. Drop uses undo toast, no confirmation.
11. No same-day Reconcile re-trigger after skip.
12. After skipped reconcile, use Option A: no banner, no separate page, reappear later.
13. No stored DailyRollup table in Phase 1.
14. Carried is not a Task status.
15. Event metadata stays metric-driven.
16. Capture `carryCountAtEvent` on `task_dropped` as well as `task_carried`.

---

# Remaining Open Questions

These are still open before formal specs:

## Day-0 Onboarding

1. Should Day-0 include the vague one-line Reconcile preview, or should the user discover Reconcile only when needed?
2. Is a soft target like "reach a usable planning surface quickly" enough, or should the team set a measurable onboarding-time target after prototype testing?

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

## Technical Foundation / Stack

This should now be addressed before formal Day-0 / Reconcile / Data Model specs.

Open stack decisions:

1. Frontend: React + Vite or Next.js?
2. Backend: Spring Boot or another backend stack?
3. Database: PostgreSQL or something else?
4. Auth: simple test auth, email/password, magic link, or OAuth?
5. Deployment: VPS/Docker, managed platform, or other?
6. PWA: basic installable PWA now, offline-first later?
7. Event log architecture: append-only table shape and API boundary?
8. AI readiness: what exactly belongs in Phase 1 without runtime LLM implementation?

---

# Updated GPT Suggested Sequence

```txt
1. Claude confirms the final Day-0 answers and remaining concerns
2. Create Technical Foundation / Stack Decision
3. Review stack with Claude/backend developer
4. Create Day-0 Onboarding Spec
5. Create Reconcile UX Spec
6. Create Data Model Phase 1 Spec
7. Review with Claude/backend/designer
8. Mahdi approves
9. Start implementation planning
```

## Why this order

Technical foundation defines the build surface.

Day-0 defines the first user experience.

Reconcile defines the core behavior being tested.

Data model defines what must be built to support both.

This order reduces premature backend complexity while keeping implementation grounded.

## Claude Review Request

Please review the newly resolved Day-0 answers and the updated sequence:

1. Do you agree that task count should not be forced and the user can add any number of tasks?
2. Do you agree standalone tasks should be allowed?
3. Do you agree nothing should be forced in Day-0, including goal creation or planning a task for Today?
4. Should Day-0 include a vague one-line Reconcile preview, or should Reconcile be discovered only when needed?
5. Should onboarding time remain a soft target until prototype testing?
6. Do you agree Technical Foundation / Stack Decision should happen before the three formal specs?
7. What should be included or excluded from the Technical Foundation decision?
