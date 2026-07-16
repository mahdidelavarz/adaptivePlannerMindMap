# Discussion 017 — AI Reconcile Intelligence and Actions

## Status
Open for GPT × Claude review.

## Scope
Define what AI and deterministic rules do inside Reconcile, which suggestions exist, and which decisions remain user-owned.

## Out of Scope
- trigger/severity rules
- provider/runtime architecture
- database schema
- validation thresholds

## Questions
1. Which facts are resolved by rules rather than AI?
2. Which grouping uses structured identifiers and which uses semantic similarity?
3. How are likely duplicates identified and explained?
4. What makes a Task a stale candidate rather than a protected item?
5. How are hard deadlines and high-risk items protected?
6. What review-priority logic is allowed?
7. Which bulk suggestions may AI produce?
8. When may AI escalate from Task review to Routine, Goal, or plan review?
9. Which action set is exposed: Done, Missed, Keep, Replan, Backlog, Drop, Review Routine, Review Goal?
10. Which actions require item-level review rather than group approval?
11. How can the user split, merge, or reject an AI group?
12. How are rationale and qualitative confidence shown?
13. What minimum adaptive replanning may follow Reconcile?

## Governing Pipeline
```txt
Rules establish known facts
→ structured grouping handles obvious relations
→ AI interprets ambiguous meaning
→ user approves consequential actions
```

## Expected Decisions
- rule/AI boundary
- grouping model
- recommendation types
- action permissions
- escalation conditions
- correction UX requirements

## Dependencies
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/016-reconcile-trigger-and-severity]]

## Resolution Template
- Deterministic rules:
- AI responsibilities:
- Allowed suggestions/actions:
- Escalation rules:
- Mind map changes:
- Specs/ADRs affected:
