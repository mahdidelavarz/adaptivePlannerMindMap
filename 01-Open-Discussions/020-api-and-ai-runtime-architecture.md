# Discussion 020 — API and AI Runtime Architecture

## Status

Split into three focused discussions.

This file is the hub for the Discussion 020 runtime architecture program.

## Program Goal

Translate the accepted product, persistence, transaction, privacy, and observability contracts from Discussions 012–019 into a coherent runtime and API architecture without reopening product semantics.

## Parts

### [[01-Open-Discussions/020a-ai-runtime-boundaries-and-orchestration]]

Defines:

- application-layer AI ports
- Spring AI and provider adapter boundaries
- Planning AI and Reconcile AI orchestration
- context builders and data minimization
- synchronous versus streamed behavior
- cancellation semantics
- kill switches and degraded modes
- provider fallback boundary
- separation between AI output and canonical mutation

### [[01-Open-Discussions/020b-api-and-frontend-state-contracts]]

Defines:

- API resources and endpoint families
- PlanningDraft and ReconcileSession contracts
- confirmation and command submission
- expected versions and idempotency keys
- conflict, retry, cancellation, and refresh behavior
- frontend state machines
- polling or streaming transport contracts

### [[01-Open-Discussions/020c-structured-output-reliability-and-cost-controls]]

Defines:

- structured-output parsing and validation
- deterministic repair allowlist
- retry classification
- timeouts, circuit breakers, and rate limits
- prompt and model version registry
- token and cost budgets
- provider operational observability
- fallback and rollout controls

## Locked Inputs

The Discussion 020 parts must preserve:

- no AI authority over canonical mutation
- complete structured PlanningDraft output only
- no usable partial output in MVP
- Reconcile AI receives structured facts only and no free-text context
- explicit confirmation before consequential mutation
- revalidate-before-commit
- transaction and idempotency contracts from Discussion 019B
- observability, retention, and access boundaries from Discussion 019C
- fail closed for AI mutation and fail open for manual product access

## Sequence

```txt
020A runtime boundaries and orchestration
→ 020B API and frontend state contracts
→ 020C structured output, reliability, and cost controls
→ 021 validation plan and release gates
→ 022 implementation sequence
```

## Out of Scope

- reopening accepted product behavior
- physical database schema
- metric thresholds
- milestone sequencing
- UI visual design
- multi-region active-active architecture

## Program Closure Conditions

Discussion 020 closes only after 020A, 020B, and 020C are accepted and their Mind Map impacts are recorded.