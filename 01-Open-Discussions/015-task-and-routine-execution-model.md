# Discussion 015 — Task and Routine Execution Model

## Status
Open for GPT × Claude review.

## Scope
Define how Tasks, Projects, Routines, RoutineOccurrences, and Today behave during normal execution.

This discussion consumes the conceptual model accepted in [[01-Open-Discussions/012-core-product-model]] and resolves execution semantics only. It must not redefine the core entities or their ownership relationships.

## Out of Scope
- AI planning conversation
- AI planning output schema
- Reconcile grouping intelligence
- database schema and API details
- provider/runtime architecture
- validation thresholds

## Questions
1. How is Today assembled from standalone Tasks, Project Tasks, standalone Routines, and Project Routines?
2. When does a Task enter Today?
3. When and how are RoutineOccurrences generated or derived?
4. What happens when the app is not opened for one or more scheduled dates?
5. Which execution transitions exist for Task, Project, Routine, and RoutineOccurrence?
6. Is a past RoutineOccurrence marked Done, Missed, Skipped, or another state?
7. Can a RoutineOccurrence ever be Carried? Current invariant: no.
8. How do stop and edit work for a Routine? Is pause/resume excluded from the first version?
9. Does editing recurrence affect only future occurrences, and from which effective date?
10. Can users complete or correct a past occurrence retroactively?
11. How are timezone and local-date boundaries handled?
12. What unresolved execution state becomes eligible for Reconcile?
13. When a Project becomes `COMPLETED`, at what execution boundary do its active Routines become `STOPPED`?
14. Does `Project ACTIVE → STOPPED` also stop all active child Routines? The core model strongly implies yes, but this execution rule must be explicit.
15. What happens to a RoutineOccurrence scheduled for the same local date on which its Project is completed or stopped?
16. What happens to already-known future occurrences after a Project-owned Routine is automatically stopped?
17. What happens to active or unresolved Tasks when their Project is marked `COMPLETED` or `STOPPED`?
18. Can completed, stopped, dropped, or missed entities be reopened or corrected in the first version?
19. Must Project completion be blocked while active Tasks remain, or may the user complete the Project and resolve those Tasks separately?

## Accepted Inputs from Discussion 012
These are not reopened here:

```txt
Task never links directly to Goal.
Routine never links directly to Goal.
Task and Routine belong to at most one Project or remain standalone.
A Project Routine is specific to that Project.
Project COMPLETED automatically stops active child Routines.
Historical RoutineOccurrences remain history.
Carry is a Task transition, not a Task status.
RoutineOccurrence is never Carried.
```

## Expected Decisions
- Today generation rules
- Task execution lifecycle
- Project completion/stop execution semantics
- Routine lifecycle and edit semantics
- occurrence lifecycle and correction rules
- project-to-routine shutdown behavior
- handling of unresolved child work
- timezone rules

## Dependencies
Requires [[01-Open-Discussions/012-core-product-model]].

## Resolution Template
- Today rules:
- Task transitions:
- Project execution transitions:
- Routine transitions:
- Occurrence transitions:
- Project completion/stop cascade behavior:
- Timezone and correction rules:
- Deferred implementation questions:
- Mind map changes:
- Specs/ADRs affected:
