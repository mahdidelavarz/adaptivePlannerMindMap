# Discussion 015 — Task and Routine Execution Model

## Status
Open for GPT × Claude review.

## Scope
Define how Tasks, Routines, RoutineOccurrences, and Today behave during normal execution.

## Out of Scope
- AI planning conversation
- Reconcile grouping intelligence
- database schema and API details

## Questions
1. How is Today assembled?
2. When does a Task enter Today?
3. When and how are RoutineOccurrences generated?
4. What happens if the app is not opened?
5. Which statuses exist for Task, Routine, and RoutineOccurrence?
6. Is a past RoutineOccurrence Done, Missed, Skipped, or another state?
7. Can a RoutineOccurrence ever be Carried? Current proposal: no.
8. How do pause, resume, stop, and edit work for a Routine?
9. Does editing recurrence affect past or future occurrences?
10. Can users complete or correct a past occurrence retroactively?
11. How are timezone and localDate boundaries handled?
12. What unresolved state is eligible for Reconcile?

## Expected Decisions
- Today generation rules
- Task lifecycle
- Routine lifecycle
- occurrence lifecycle
- recurrence-edit semantics
- timezone rules

## Dependencies
Requires [[01-Open-Discussions/012-core-product-model]].

## Resolution Template
- Today rules:
- Task states/transitions:
- Routine states/transitions:
- Occurrence states/transitions:
- Mind map changes:
- Specs/ADRs affected:
