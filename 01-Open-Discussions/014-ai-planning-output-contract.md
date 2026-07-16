# Discussion 014 — AI Planning Output Contract

## Status
Open for GPT × Claude review.

## Scope
Define the structured, bounded draft produced by Planning AI before user approval.

## Out of Scope
- conversation UI
- persistence schema
- provider/runtime selection
- Reconcile output

## Questions
1. What exact fields exist in the planning draft?
2. Does AI create or only clarify the Goal?
3. What are the maximum Task and Routine counts?
4. Is a seven-day schedule required or optional?
5. Which Tasks receive planned dates?
6. Which recurrence patterns are supported?
7. Are assumptions, rationale, and warnings included?
8. Can the AI intentionally return zero Tasks or zero Routines?
9. How are invalid dates, unsupported recurrence, and duplicate items handled?
10. How is structured output validated and repaired?
11. What may the user edit before confirmation?
12. Is the whole draft confirmed together or item by item?

## Candidate Shape
```txt
PlanningDraft
- clarifiedGoal?
- assumptions[]
- tasks[]
- routines[]
- firstWeekSchedule[]
- warnings[]
```

## Expected Decisions
- canonical draft contract
- numeric limits
- validation rules
- user-review semantics
- no-output and partial-output behavior

## Dependencies
- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]

## Resolution Template
- Accepted output contract:
- Limits:
- Validation/fallback rules:
- Mind map changes:
- Specs/ADRs affected:
