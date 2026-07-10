# Week-1 Evaluation Model

## Purpose

Define how the product evaluates the first week without scoring the user like a productivity police officer with a spreadsheet.

The model should evaluate **plan-user fit**, not human worth or discipline.

## Core Principle

Week-1 evaluation should not ask:

> Was the user good or bad?

It should ask:

> Was the plan sized, timed, and shaped correctly for this user's actual behavior?

## Inputs

- tasks created
- tasks planned
- tasks completed
- tasks carried
- tasks dropped
- app opens
- reconcile starts
- reconcile completions
- weekly review completion
- first action latency
- return after missed day

## Early Signals

### Plan too large

Possible signs:

- many carried tasks
- low completion
- repeated task shrinking
- user returns but keeps carrying the same work

### Tasks too vague

Possible signs:

- repeated carry-forward
- high first action latency
- user edits or splits tasks after failure

### Good restart behavior

Possible signs:

- user misses a day but returns
- user completes reconcile after unresolved tasks
- user drops or shrinks instead of carrying everything blindly

### Weak core loop

Possible signs:

- user opens app but avoids reconcile
- user abandons after unresolved tasks appear
- user creates tasks but does not return

## Draft Evaluation Output

The weekly review should produce a simple, non-judgmental summary:

```txt
This plan may have been too large for the available week.
You returned after missing tasks, which is a strong restart signal.
Several tasks were carried forward more than once, so next week should use smaller task slices.
```

## What Not To Do

- No productivity score
- No failure score
- No streak ranking
- No moral judgment
- No fake precision

## Open Questions

- Should the first evaluation be after 7 calendar days or 7 active days?
- Should the system show a confidence level next to each insight?
- What minimum data is required before showing any pattern?
- Should AI be allowed to propose changes in Week 1, or only summarize?
