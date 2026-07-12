# Day-0 Onboarding

## Status

Draft approved on agreed decisions. Remaining sequencing question is tracked separately.

## Purpose

Define the user's first experience without forcing a rigid setup ritual or contaminating Phase 1 validation.

## Standing Product Principle

```txt
User input is free.
System suggestions are gated.
```

This principle applies beyond Day-0:

- users may create any number of tasks
- tasks may exist independently
- goals are optional
- planning a task for Today is optional
- the system should only make personalized suggestions after enough behavioral evidence exists

The product should guide, not coerce. Apparently software can survive without behaving like a disappointed school principal.

## Day-0 Goals

Day-0 should help the user:

1. understand the basic planning surface
2. create real tasks with minimal friction
3. optionally connect tasks to goals
4. optionally plan tasks for Today / Tomorrow / Later
5. reach a usable app state quickly

## Task Entry

There is no required task count.

The user may create:

- one task
- several tasks
- many tasks

Suggested guidance:

```txt
Add a few tasks to get started. You can add more later.
```

This is guidance only, not a validation rule or UI restriction.

## Optional Goals

A task may be:

- standalone
- optionally linked to a goal

Goal creation is not mandatory during onboarding.

The data model must allow:

```txt
goalId = null
```

## Optional Planning

Planning a task for Today is not mandatory.

The product may offer a guided path:

```txt
Want to start today? Pick one task for Today.
```

But the user can continue without doing so.

## First-Class Empty Today Path

Because nothing is forced, some users will finish onboarding with no tasks planned for Today.

This is not an edge case. It is a supported path that needs deliberate design.

The Today empty state should:

- remain calm and non-judgmental
- explain that nothing is planned
- allow adding a new task
- allow viewing or planning unscheduled tasks
- avoid implying failure or incompleteness

Example:

```txt
Nothing planned for today.

You can add something now, or leave today open.
```

Possible actions:

```txt
[Add a task]
[View unscheduled tasks]
```

## Reconcile Preview Decision

Do not use emotional Reconcile promises during Day-0.

Avoid copy such as:

```txt
We help you recover without guilt or punishment.
```

Reason:

Phase 1 qualitative validation includes spontaneous reports such as:

- less guilt
- easier restart
- clearer next action

Priming users with that language can create demand-characteristics bias and inflate the same signal the test is supposed to observe.

If any preview is used, keep it mechanical and neutral:

```txt
Your plan can adjust as things change.
```

Do not explain Done / Carry / Drop during onboarding.

## Onboarding Time

Do not lock a hard 3–5 minute requirement before prototype testing.

Use a soft objective:

```txt
The user should reach a usable planning surface quickly.
```

Measure actual onboarding time during prototype testing, then set a threshold from observed behavior rather than ceremonial optimism.

## Proposed Flow

```txt
1. Welcome
   Brief, neutral product introduction.

2. Optional goal
   User may create a goal or skip.

3. Task entry
   User adds any number of real tasks.

4. Optional planning
   User may assign Today / Tomorrow / Later or leave tasks unscheduled.

5. Usable planning surface
   - Today with planned tasks, or
   - calm empty Today with actions to add or plan work.
```

## Phase 1 AI Boundary

Phase 1 is AI-ready, not AI-implemented.

Keep:

- source field
- `ai_generated` enum value
- AI architecture notes
- Phase 2 AI planning in the map

Do not implement:

- runtime LLM integration
- prompt execution
- AI planning endpoints
- hidden AI feature flags
- AI Knowledge meter UI

## Designer Notes

Designer should cover both common Day-0 outcomes:

1. user plans work for Today
2. user plans nothing and reaches an empty Today state

Focus on:

- low-friction task entry
- optional goal linking
- calm empty states
- guidance without force
- clear next actions
- no shame or productivity scoring

## Backend Requirements

Backend must support:

- standalone task creation
- optional `goalId`
- optional `plannedForDate`
- manual source tagging
- onboarding events
- unscheduled-task retrieval

## Explicitly Excluded

- forced goal creation
- forced task count
- forced Today planning
- personality quiz
- long questionnaire
- emotional Reconcile priming
- runtime AI
- routines
- time-blocking
- deadlines
- productivity score
