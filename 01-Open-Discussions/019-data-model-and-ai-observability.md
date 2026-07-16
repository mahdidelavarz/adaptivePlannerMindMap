# Discussion 019 — Data Model and AI Observability

## Status
Open for GPT × Claude review.

## Scope
Translate accepted product behavior into the minimum honest persistence model and event history required to evaluate AI and execution.

## Out of Scope
- provider/runtime choice
- endpoint design
- validation thresholds
- implementation sequencing

## Questions
1. What are the final fields and invariants for Goal, Task, Routine, and RoutineOccurrence?
2. How is `source: manual | ai_assisted` represented?
3. Does the PlanningDraft remain fully ephemeral until approval?
4. Is any AI interaction or proposal entity required?
5. How are accepted, edited, rejected, and ignored suggestions recorded?
6. How are AI Reconcile groups and affected item IDs represented?
7. Is a ReconcileSession entity now necessary, or can events still derive sessions safely?
8. Which raw prompt/response data is forbidden, optional, or required to persist?
9. What event taxonomy covers planning, execution, and Reconcile?
10. What metadata is required for severity and descriptive analysis?
11. Which database constraints enforce lifecycle correctness?
12. What are transaction and idempotency boundaries for approved bulk actions?
13. How are model/prompt versions recorded without coupling domain tables to a provider?

## Expected Decisions
- ERD
- field definitions
- lifecycle invariants
- event taxonomy
- AI observability boundary
- persistence and transaction rules

## Dependencies
Requires accepted product behavior from Discussions 012–018.

## Resolution Template
- Models:
- Invariants:
- Event taxonomy:
- AI observability:
- Session/idempotency decisions:
- Mind map changes:
- Specs/ADRs affected:
