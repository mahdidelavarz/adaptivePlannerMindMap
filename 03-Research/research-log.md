# Research Log

## Purpose

Track evidence behind the Adaptive Planner assumptions.

Research should not be dumped into the project like a pile of PDFs nobody reads. Each source needs a claim, relevance, and confidence label.

## Confidence Labels

| Label | Meaning |
|---|---|
| Verified | Strong enough to support a product assumption |
| Useful but unverified | Interesting, but needs more validation |
| Theoretical support | Helpful framing, not direct proof |
| Anecdotal but direct | Real user-language evidence, useful for problem discovery but not scientific proof |
| Internal model | Product reasoning model that still needs external support |

## Current Research Themes

### 1. Zeigarnik Effect

Claim:

Unfinished tasks can remain mentally active and create cognitive load.

Relevance:

Supports the idea that unresolved tasks can create mental friction and return avoidance.

Confidence:

Theoretical support.

### 2. Rumination and Sleep Disruption

Claim:

Unfinished goals and task-related rumination may interfere with sleep or mental recovery.

Relevance:

Supports the emotional weight of unfinished tasks.

Confidence:

Useful but unverified for this product.

### 3. Planner Abandonment

Claim:

Users often abandon task tools when overdue lists become emotionally heavy or cluttered.

Relevance:

Directly supports the core pain hypothesis.

Confidence:

Anecdotal but direct. Needs user interviews/test data before becoming a validated product assumption.

Related file:

- [[06-References/social-evidence]]

### 4. Existing Tool Patterns

Observation:

Some tools preserve past tasks and overdue views in ways that may help some users but shame or overwhelm others.

Relevance:

Helps position reconcile-on-open as a restart mechanism.

Confidence:

Useful but unverified.

### 5. Habit Formation — Lally et al.

Source:

Lally et al. — *How are habits formed: Modelling habit formation in the real world*.

Link:

https://doi.org/10.1002/ejsp.674

Claim:

Habit formation varies substantially between people and usually takes longer than simplistic "21 days" claims.

Relevance:

The MVP should not overvalue short streaks or assume stable behavior after a few days. Adaptive planning should treat habit formation as gradual and variable.

Confidence:

Useful but unverified for this product until tested with target users.

### 6. Confidence Scoring Framework

Claim:

The product should progressively increase confidence as it observes repeated, stable behavior patterns.

Relevance:

Supports the AI Knowledge Level page and gated AI suggestions.

Confidence:

Internal model. Needs external support and product validation.

Working idea:

```txt
single signal → clue
repeated signal → weak pattern
stable repeated signal → stronger pattern
actionable pattern → suggestion allowed
```

This should not be presented as a verified scientific model until sources and validation are added.

## Needed Research

- User interviews with people who abandoned Notion/Todoist/TickTick.
- Evidence around restart behavior after failure.
- Research on self-compassion vs shame in behavior change.
- Evidence on implementation intentions.
- Evidence on planning fallacy and task breakdown.
- External support for progressive confidence scoring.
- Product examples that expose AI/user-model confidence transparently.

## Research Entry Template

```md
# Source Title

## Link

## Claim

## Relevance to Product

## Confidence Label

## Notes
```
