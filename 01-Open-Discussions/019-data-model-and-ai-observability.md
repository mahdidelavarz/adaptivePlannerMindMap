# Discussion 019 — Data Model and AI Observability

## Status
Open for GPT × Claude review.

## Scope
Translate accepted product behavior into the minimum honest persistence model and event history required to evaluate AI and execution.

This discussion receives conceptual entities and lifecycle rules from Discussions 012–018. It must not reopen whether Goal, Project, Task, Routine, or RoutineOccurrence exist; it decides how accepted concepts are represented safely.

## Out of Scope
- provider/runtime choice
- endpoint design
- validation thresholds
- implementation sequencing
- redefining product ownership rules accepted in Discussion 012

## Questions
1. What are the final fields and invariants for Goal, Project, Task, Routine, and RoutineOccurrence?
2. How are nullable ownership relationships represented while enforcing these rules?
   - `Project → Goal?`
   - `Task → Project?`
   - `Routine → Project?`
   - no `Task → Goal`
   - no `Routine → Goal`
3. Which database constraints ensure a Task or Routine belongs to at most one Project and prevent cross-Project Routine reuse?
4. How is `source: manual | ai_assisted` represented?
5. Does the PlanningDraft remain fully ephemeral until approval?
6. Is any AI interaction or proposal entity required?
7. How are accepted, edited, rejected, and ignored suggestions recorded?
8. How are AI Reconcile groups and affected item IDs represented?
9. Is a ReconcileSession entity necessary, or can events derive sessions safely?
10. Which raw prompt/response data is forbidden, optional, or required to persist?
11. What event taxonomy covers planning, execution, Project lifecycle, Routine shutdown, and Reconcile?
12. What metadata is required for severity and descriptive analysis?
13. Which database constraints enforce lifecycle correctness?
14. What is the transaction boundary for `Project ACTIVE → COMPLETED` plus automatic stopping of all active child Routines?
15. What is the transaction boundary for `Project ACTIVE → STOPPED` if Discussion 015 confirms the same Routine shutdown invariant?
16. How are already-persisted future RoutineOccurrences handled when the owning Routine is stopped?
17. How are concurrent operations handled when Project completion, Routine editing, occurrence completion, or Task updates happen at nearly the same time?
18. What idempotency guarantees prevent duplicate cascade effects or duplicate AI-approved mutations?
19. Should Project completion/stop and child Routine shutdown create one domain event, multiple events, or both?
20. How is historical identity preserved when a similar Routine is later created under another Project?
21. How are model and prompt versions recorded without coupling domain tables to a provider?
22. Which derived summaries may be computed from execution data without being persisted as misleading Goal progress?

## Accepted Inputs from Earlier Discussions
The data model must preserve these semantic rules:

```txt
Goal may own zero or more Projects.
Project may be standalone.
Task and Routine may be standalone.
Task and Routine never link directly to Goal.
A Routine belongs to one Project or is standalone.
A Project Routine is not moved or reused automatically.
Project completion stops active child Routines.
Historical RoutineOccurrences remain preserved.
Goal achievement is user-confirmed, not inferred from execution totals.
Plan is not a persisted domain entity.
```

## Expected Decisions
- ERD
- field definitions
- ownership constraints
- lifecycle invariants
- Project-to-Routine cascade persistence
- event taxonomy
- AI observability boundary
- persistence, concurrency, transaction, and idempotency rules
- derived-summary versus persisted-truth boundary

## Dependencies
Requires accepted product behavior from Discussions 012–018.

## Resolution Template
- Models:
- Relationships and constraints:
- Lifecycle invariants:
- Project cascade transaction:
- Concurrency/idempotency:
- Event taxonomy:
- AI observability:
- Session decisions:
- Derived summaries:
- Mind map changes:
- Specs/ADRs affected:
