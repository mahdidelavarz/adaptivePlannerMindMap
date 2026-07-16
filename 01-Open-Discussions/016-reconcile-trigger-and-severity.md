# Discussion 016 — Reconcile Trigger and Severity

## Status
Open for GPT × Claude review.

## Scope
Define when Reconcile appears and how light daily deviation differs from true recovery after absence or backlog growth.

## Out of Scope
- semantic grouping and AI recommendations
- visual design details
- persistence and API implementation

## Questions
1. Is one unresolved Task from yesterday enough to trigger Reconcile?
2. Does absence create a different entry path or only a higher severity?
3. When must Reconcile not be shown?
4. Can the user Skip and enter Today directly?
5. How often can Reconcile trigger in one local day?
6. Which inputs determine severity: absenceDays, count, age, deadlines, Routine occurrences, Carry count?
7. What are the exact Light, Medium, and Recovery thresholds?
8. Is severity calculated before or after rule-based cleanup of Routine occurrences?
9. Does Reconcile block Today or remain optional?
10. What happens when new overdue work appears after the flow is completed?
11. How is same-day retrigger prevented?
12. Which context fields must be recorded for validation?

## Candidate Bands
```txt
Light — small one-day deviation
Medium — several days or moderate unresolved work
Recovery — meaningful absence or large/stale backlog
```

## Expected Decisions
- eligibility rule
- trigger timing
- severity formula
- skip/retrigger behavior
- context metadata

## Dependencies
Requires [[01-Open-Discussions/015-task-and-routine-execution-model]].

## Resolution Template
- Eligibility:
- Severity thresholds:
- Skip/retrigger rules:
- Context fields:
- Mind map changes:
- Specs/ADRs affected:
