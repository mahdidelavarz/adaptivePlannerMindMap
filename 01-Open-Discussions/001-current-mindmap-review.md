# Discussion 001 — Current Mind Map Review

## Status
Reviewed by Claude — partially applied, remaining issues moved to Discussion 002

## Context

This discussion mirrors the first GPT review section that was added to FigJam.

The goal was to review the current Adaptive Planner mind map and identify missing sections, weak assumptions, and unclear decisions before implementation starts.

The repo is now further ahead than the original review assumed. Several sections that were originally missing now exist as real spec files.

## GPT Draft

### Verdict

The map is strong enough to align a 3-person team. It has the right spine:

- vision
- hypothesis
- locked decisions
- risks
- data
- validation
- ownership
- flow

It is stronger than vague "AI-powered startup" fog.

### Strengths

- Reconcile-on-open is clearly the core loop.
- Locked decisions are explicit:
  - no shame UI
  - no streak-first design
  - append-only events
  - AI draft-only
- Risks are called out, especially the AI-planning confound.
- Team ownership is useful for Mahdi, designer, and backend developer.
- The MVP flow is concrete enough to discuss.

### Original Gaps / Problems

1. Target Segment + JTBD is too soft and needs a dedicated block.
2. MVP boundary is not sharp enough.
3. Day-0 onboarding is missing.
4. AI Guardrails need a standalone section.
5. Data Events need a concrete taxonomy.
6. Metrics need success thresholds.
7. Research log needs confidence labels.
8. Week-1 evaluation formula is a product decision, not just math.

## Claude Review

Claude agreed with the overall verdict but noted that the repo is now further ahead than this discussion file assumed.

Claude's key points:

1. MVP Scope Boundary, AI Guardrails, Event Taxonomy, Metric Thresholds, and Research Confidence already exist as real files.
2. The review should now focus on completeness, not existence.
3. `mvp-scope.md` includes AI-generated 7-day draft plans, which may confound a clean test of the reconcile hypothesis.
4. Still genuinely missing:
   - Target Segment + JTBD
   - Day-0 Onboarding
   - Week-1 Evaluation Formula / Model
5. `ai-guardrails.md` was missing a sensitive-topics restriction.
6. `event-taxonomy.md` may need to distinguish task creation from task scheduling.
7. `research-log.md` was missing habit-formation research and a clearer confidence/social-evidence structure.

## GPT Analysis of Claude Review

Claude is mostly right.

Changes applied directly:

- Target Segment + JTBD created.
- Week-1 Evaluation Model created.
- Sensitive-topic guardrails added.
- Lally habit-formation research added.
- Social Evidence reference file added.
- Research confidence labels refined.

Changes not directly locked yet:

- AI planning vs reconcile confound.
- Whether AI-generated planning should be in MVP, behind a flag, or part of a two-loop MVP.
- AI Knowledge Level page.
- Task scheduling model.
- Progressive Bayesian Confidence source/position.

These have been moved to:

- [[01-Open-Discussions/002-ai-planning-reconcile-and-scheduling]]

## Applied Additions to Mind Map

- Target Segment + JTBD
- Week-1 Evaluation Model
- Social Evidence
- AI Planning / Reconcile Discussion

## Still Open

- Day-0 Onboarding needs its own artifact.
- AI planning and reconcile need a validation strategy.
- Scheduling model needs a product decision.
- AI Knowledge Level page needs Claude review.
- Progressive Bayesian Confidence needs source clarification.

## Final Decision

Partially applied. Remaining open questions moved to Discussion 002.
