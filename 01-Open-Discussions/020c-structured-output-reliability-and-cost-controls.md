# Discussion 020C — Structured Output, Reliability, and Cost Controls

## Status

Open after Discussion 020A stabilizes.

## Scope

Define how AI output is validated, rejected, retried, rate-limited, versioned, and cost-controlled without weakening accepted product, privacy, or transaction guarantees.

## Required Inputs

- final Discussion 020A runtime boundaries
- final Discussion 020B API state contracts
- Discussion 018C failure and privacy resolution
- Discussion 019C AI operational observability and retention resolution

## Questions

1. What exact validation pipeline applies before an AI result becomes a usable proposal?
2. Which deterministic repairs are allowlisted?
3. Which failures are retryable and which are final?
4. How many provider retries are permitted per operation?
5. What timeout layers exist for connection, first token, and total operation time?
6. When does a circuit breaker open and what degraded behavior follows?
7. Which rate limits apply per user, operation, provider, and system?
8. Is provider fallback permitted in the pilot?
9. How are prompt templates, schemas, rule catalogs, and model configurations versioned?
10. What token budgets exist for Planning and Reconcile?
11. How is context truncated without silently changing meaning?
12. What cost ceilings stop repeated or oversized operations?
13. Which operational fields are recorded without retaining raw content?
14. How are kill switches tested and audited?
15. What user-facing message corresponds to each flow-affecting failure class?

## Locked Inputs

- partial output is rejected entirely in MVP
- deterministic repair cannot invent semantic intent
- reviewDate is never freely AI-generated
- Reconcile AI receives no free-text context
- retry never authorizes mutation
- raw prompt and response content is restricted and short-lived by default
- provider failure leaves canonical state unchanged
- manual and deterministic product paths remain available

## Expected Decisions

- structured-output pipeline
- repair allowlist
- retry matrix
- timeout and circuit-breaker policy
- rate-limit policy
- provider fallback policy
- prompt/model/schema version registry
- token and cost budgets
- operational observability fields
- kill-switch runtime behavior
- Mind Map impact

## Out of Scope

- endpoint naming
- canonical transaction logic
- provider contract details already settled in 020A
- product metric thresholds
- deployment sequencing
