# ADR-001 — Reconcile-on-open is the MVP Core Loop

## Status
Locked for MVP draft

## Context

The project initially had a large long-term vision: an adaptive personal planning and learning system that understands the user over time.

That vision is too large for the first implementation.

The MVP needs a narrow behavioral loop.

## Decision

The first MVP will use **reconcile-on-open** as its core loop.

When the user opens the app and has unresolved tasks, the app first helps them reconcile the past before planning the future.

## User Actions

- Done
- Carry
- Drop

## Why

The core hypothesis is that unfinished work creates guilt, avoidance, and abandonment.

A planner that helps users restart after slippage may create more long-term value than a planner that simply piles up overdue tasks.

## Consequences

### Positive

- Clear MVP focus
- Strong behavioral hypothesis
- Differentiates from basic to-do lists
- Creates useful data about completion, avoidance, and carry-forward behavior

### Negative

- May feel like friction if implemented badly
- Needs very careful non-judgmental UX
- Success depends on return behavior, not just task completion

## Related Files

- [[04-Specs/mvp-scope]]
- [[04-Specs/event-taxonomy]]
- [[04-Specs/metrics]]
- [[05-Flows/mvp-core-loop]]
