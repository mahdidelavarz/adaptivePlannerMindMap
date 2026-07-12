# Reconcile UX — Phase 1

## Status

Formalized from Discussion 003 and Claude review.

## Purpose

Help users resolve unfinished planned work without shame and return to Today.

Reconcile is the core Phase 1 behavior being validated.

## Trigger

Show Reconcile when:

- the user opens the app, and
- unresolved planned-for-day tasks exist from a previous day

## Presentation

Use a lightweight focused panel or page.

Do not use an aggressive modal or a blocking dead-end.

The exact placement relative to Today remains a design implementation decision, but the flow must feel calm and skippable.

## Suggested Copy

```txt
You have 3 unfinished tasks from earlier.
Let's clean things up so today stays clear.
```

Avoid:

- red overdue labels
- failure language
- guilt framing
- productivity scores

## Actions

For each unresolved task:

```txt
[Done] [Carry] [Drop]
```

The full Reconcile flow also provides:

```txt
[Not now]
```

`Not now` is preferred over `Skip for now` because it reads as a temporary choice without sounding like a test-control label escaped into the interface.

## Done

Meaning:

The task was completed outside the app.

Result:

```txt
status = completed
```

Event:

```txt
task_completed
```

## Carry

Meaning:

Keep the task active and assign a new planned day.

Default:

```txt
Today
```

User can change with one tap:

```txt
Today / Tomorrow / Later
```

Do not force a separate date choice for every task.

A carried task remains:

```txt
status = active
```

Carry changes:

```txt
plannedForDate
carryCount
```

Event:

```txt
task_carried
```

## Drop

Meaning:

The task no longer matters.

Do not:

- ask for a reason in Phase 1
- show a confirmation dialog

Use an undo toast:

```txt
Dropped. Undo
```

Result:

```txt
status = dropped
```

Event:

```txt
task_dropped
```

## Skipped Reconcile

The correct term is **skipped reconcile**, not skipped tasks.

When the user chooses `Not now`:

```txt
reconcile_skipped
```

Phase 1 uses Option A:

```txt
Skipped reconcile
→ no persistent Today banner
→ no separate unresolved page
→ no same-day re-trigger
→ unresolved tasks remain unresolved
→ Reconcile may appear again on the next app open on a later day
```

Reason:

This has the lowest build cost and preserves the cleanest validation signal.

A second banner or review page would create another route back to unresolved tasks, making it harder to know whether post-skip behavior was organic or prompted by a second UI surface.

If testers report that unresolved tasks feel lost, a calm review entry point can be added later based on evidence.

## Post-Skip Measurement

If the user skips Reconcile and takes a meaningful action within 5 minutes, derive or log:

```txt
meaningful_action_after_reconcile_skip
```

Meaningful actions include:

- completing a task
- carrying a task
- dropping a task
- creating a task
- planning an unscheduled task
- completing a weekly review

## Re-trigger Rule

After skipped reconcile:

- do not show Reconcile again on the same calendar day
- allow it to appear on a later app open after the day changes
- only show it if unresolved planned work still exists

## Completion State

After Reconcile completes:

```txt
Today is clean.
Here is what you chose to keep.
```

Then route to Today.

## Required Events

```txt
reconcile_shown
reconcile_started
reconcile_skipped
reconcile_completed
task_completed
task_carried
task_dropped
```

## Required Metadata

### `reconcile_shown`

```txt
unresolvedTaskCountShown
```

### `reconcile_skipped`

```txt
unresolvedTaskCountShown
```

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

Capturing carry count on drop supports later pattern detection such as:

```txt
task carried 3 times, then dropped
```

## Explicitly Excluded

- forced Reconcile
- same-day nagging after skip
- persistent unresolved-task banner
- separate unresolved/backlog page
- Drop confirmation dialog
- mandatory Drop reason
- red overdue UI
- shame or failure copy
- productivity score
