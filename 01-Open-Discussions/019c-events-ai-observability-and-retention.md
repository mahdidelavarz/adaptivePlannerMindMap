# Discussion 019C — Events, AI Observability, and Retention

## Status

Open after Discussion 019A stabilizes.

## Scope

Define the minimum honest event and observability model required to audit product decisions, evaluate AI behavior, and protect user data.

This discussion will cover:

- domain events and audit events
- ReconcileSession representation
- PlanningDraft persistence decision
- AI proposal and operation records
- accepted, edited, rejected, cancelled, and ignored suggestion outcomes
- affected entity IDs and grouping
- rule IDs, versions, reason codes, observed metrics, and evidence quality
- model, prompt-template, and catalog versioning without coupling domain tables to a provider
- product event log versus AI operational log versus raw prompt/response content
- retention and redaction boundaries
- hostile-content and crisis-flow observability
- manual fallback and degraded-mode observability
- derived analytics versus persisted product truth

## Required Inputs

- final Discussion 019A schema and canonical-versus-derived boundaries
- final Discussion 019B transaction and event-publication contract
- Discussion 017B rule and recommendation contract
- Discussion 018A audit and trust contract
- Discussion 018C privacy, failure, and retention contract

## Expected Decisions

- event taxonomy
- session and proposal model
- observability field definitions
- retention classes
- provider-neutral version fields
- correlation and causation IDs
- analytics-safe derived summaries
- forbidden persistence
- minimum indexes and access boundaries
- Mind Map impact

## Deferred Until 019A Closes

No observability record may invent canonical entity semantics or duplicate mutable domain truth merely for analytics convenience.
