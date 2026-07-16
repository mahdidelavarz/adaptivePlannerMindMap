# Discussion 020 — API and AI Runtime Architecture

## Status
Open for GPT × Claude review.

## Scope
Define the technical runtime, API contracts, provider boundary, validation, and failure behavior for Planning AI and Reconcile AI.

## Out of Scope
- reopening accepted product behavior
- metric thresholds
- milestone sequencing

## Questions
1. Which LLM provider and model class are used for the pilot?
2. Is Spring AI used directly, wrapped by an internal port, or avoided?
3. Which provider-specific concepts must remain outside the domain layer?
4. Are planning and Reconcile requests synchronous, streamed, or hybrid?
5. What endpoints and request/response contracts are required?
6. How is structured output validated, repaired, retried, or rejected?
7. What are timeout, retry, circuit-breaker, and rate-limit policies?
8. What happens when AI is unavailable after the user has entered the flow?
9. How are prompt templates versioned and tested?
10. How are model and prompt versions attached to events?
11. How are token use, latency, and cost controlled?
12. What context is sent for planning versus Reconcile?
13. How are bulk mutations applied transactionally after user approval?
14. Which frontend loading, partial, error, retry, and cancellation states exist?
15. Is a provider fallback required for the pilot?

## Expected Decisions
- runtime architecture
- provider abstraction
- API contracts
- structured-output strategy
- reliability policies
- cost and observability controls
- frontend state contract

## Dependencies
Requires accepted outputs from Discussions 013, 014, 017, 018, and 019.

## Resolution Template
- Provider/runtime:
- Endpoints/contracts:
- Validation/retry:
- Failure fallback:
- Cost/observability:
- Mind map changes:
- Specs/ADRs affected:
