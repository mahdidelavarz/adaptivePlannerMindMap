# Adaptive Planner Mind Map

This vault is the product, architecture, research, and implementation source of truth for Adaptive Planner / Adaptive Life OS.

Phase 0 definition and architecture are closed. The project is now ready to begin Phase 1 implementation.

Start with:

- [[00-START-HERE]]
- [[02-Decisions/ADR-003-phase-0-closure]]
- [[05-Implementation/phase-1-plan]]

## Repository structure

- `00-Canvas/` — visual overview and mind-storming
- `01-Open-Discussions/` — active or resolved GPT × Claude review loops
- `02-Decisions/` — accepted ADRs and locked decisions
- `03-Research/` — evidence, references, and confidence notes
- `04-Specs/` — accepted product and technical specifications
- `05-Implementation/` — active build sequence, gates, and implementation planning
- `05-Flows/` — historical or supporting Mermaid/user/system flows when present
- `06-References/` — links, quotes, and source notes

## Current product frame

Adaptive Planner is a personal planning system that adapts to real behavior instead of relying on questionnaires, streaks, or guilt-heavy overdue lists.

The Phase 1 core loop is:

1. User logs in with phone OTP.
2. Day-0 setup allows optional Goal and Task creation.
3. User plans work through Tasks and Today.
4. On a later open, unresolved prior Tasks may trigger Reconcile.
5. User resolves them through Done, Carry, Drop, or Skip.
6. Behavioral events support the 14-day validation.

Phase 1 intentionally excludes AI runtime, Weekly Review, calendar integration, routines, time blocking, refresh tokens, OAuth/password login, and offline-first mutations.

## Team

The plan assumes:

```txt
Mahdi — frontend + product/spec ownership + E2E coordination
Backend developer — Java/Spring/PostgreSQL + security + transactions
Designer — UX/UI + interaction states + responsive behavior
```

This is two engineering streams and one design stream. The designer is not treated as a third developer.

## Source-of-truth rule

When documents conflict, follow:

```txt
latest accepted ADR
→ latest accepted formal spec
→ accepted implementation plan
→ resolved discussion summary
→ research or historical draft
```

Resolved discussions preserve decision history. They do not override formal specs.

FigJam is optional visual output. This repository remains the source of truth.