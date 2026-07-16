# Discussion 011 — AI-Native MVP Scope and Ordered Review Index

## Status

Open umbrella discussion.

The agreed product direction is an AI-native first version covering the complete vertical loop:

```txt
Goal / intention
→ AI-assisted Task and Routine creation
→ Today execution
→ AI-assisted Reconcile
→ user-approved adaptation
```

This file no longer attempts to resolve the full scope itself. The work has been split into bounded discussions that must be reviewed and closed in order.

Related background:

- [[01-Open-Discussions/010-ai-first-planning-and-reconcile-roadmap]]

---

## Required Review Order

1. [[01-Open-Discussions/012-core-product-model]]
   - Define Goal, Task, Routine, RoutineOccurrence, Plan, relationships, lifecycle, and exclusions.

2. [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]
   - Define the empty-state entry, clarification flow, domain allowlist, manual fallback, and exit/failure paths.

3. [[01-Open-Discussions/014-ai-planning-output-contract]]
   - Define the bounded structured draft produced by Planning AI and reviewed before persistence.

4. [[01-Open-Discussions/015-task-and-routine-execution-model]]
   - Define Today, Task execution, Routine scheduling, occurrence behavior, missed work, and timezone rules.

5. [[01-Open-Discussions/016-reconcile-trigger-and-severity]]
   - Define Reconcile eligibility, trigger timing, skip behavior, and Light / Medium / Recovery severity.

6. [[01-Open-Discussions/017-ai-reconcile-intelligence-and-actions]]
   - Define the rule/AI boundary, grouping, recommendations, bulk actions, escalation, and user correction.

7. [[01-Open-Discussions/018-ai-guardrails-trust-and-failure-handling]]
   - Define allowed, confirmation-required, and forbidden behavior, plus privacy, reversibility, and fallback rules.

8. [[01-Open-Discussions/019-data-model-and-ai-observability]]
   - Define persistence models, invariants, events, suggestion history, idempotency, and AI observability.

9. [[01-Open-Discussions/020-api-and-ai-runtime-architecture]]
   - Define provider/runtime architecture, API contracts, structured-output validation, reliability, cost, and frontend states.

10. [[01-Open-Discussions/021-validation-plan-and-decision-gates]]
    - Define formal hypotheses, metrics, thresholds, segmentation, decision matrix, and forbidden causal claims.

11. [[01-Open-Discussions/022-updated-mvp-implementation-plan]]
    - Convert resolved decisions into milestones, ownership, exit gates, testing, and pilot readiness.

---

## Discussion Rule

Each discussion must close with:

```txt
Accepted decisions
Deferred or rejected decisions
Mind map impact
Formal documents affected
```

A later discussion must not silently change an earlier accepted decision. Any conflict must reopen or explicitly supersede the source discussion.

## Formalization Rule

Discussion 011 and Discussions 012–022 do not silently override existing ADRs or specs.

After the ordered discussions are resolved, update the affected source-of-truth documents explicitly, including the mind map, ADRs, product specs, data model, API contracts, validation plan, and implementation plan.

## Next Discussion

Start with:

- [[01-Open-Discussions/012-core-product-model]]
