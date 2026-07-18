# Discussion 017 — AI Reconcile Intelligence and Actions

## Status
Open for GPT × Claude review.

## Scope
Define what AI and deterministic rules do inside Reconcile, which suggestions exist, and which decisions remain user-owned.

This discussion uses the hierarchy accepted in [[01-Open-Discussions/012-core-product-model]]:

```txt
Goal
→ Project
→ Task / Routine
→ RoutineOccurrence
```

It must not flatten Project-owned work into direct Goal-owned Tasks or Routines.

## Out of Scope
- trigger/severity rules
- provider/runtime architecture
- database schema
- validation thresholds
- execution semantics already assigned to Discussion 015

## Questions
1. Which facts are resolved by rules rather than AI?
2. Which grouping uses stable Project/Routine identifiers and which uses semantic similarity?
3. How are likely duplicates identified and explained?
4. What makes a Task a stale candidate rather than a protected item?
5. How are hard deadlines and high-risk items protected?
6. What review-priority logic is allowed?
7. Which bulk suggestions may AI produce?
8. When may AI escalate from Task review to Routine, Project, Goal, or broader plan review?
9. How does Reconcile distinguish these failure levels?
   - one stale Task
   - an unrealistic Routine
   - a blocked or ineffective Project
   - a Goal that the user may no longer pursue
10. Which action set is exposed: Done, Missed, Keep, Replan, Backlog, Drop, Review Routine, Review Project, Review Goal?
11. Which actions require item-level review rather than group approval?
12. How can the user split, merge, or reject an AI group?
13. How are rationale and qualitative confidence shown?
14. What minimum adaptive replanning may follow Reconcile?
15. May AI recommend completing or stopping a Project, or only recommend that the user review it?
16. If Project completion/stopping would automatically stop child Routines, how must that consequence be explained before confirmation?
17. When all current Projects under a Goal are completed or stopped, may AI prompt the user to confirm Goal achievement, create a new Project, or abandon the Goal?
18. How does AI avoid treating Project work completion as proof of Goal outcome achievement?
19. How are standalone Tasks and Routines reconciled when no Project context exists?

## Governing Pipeline
```txt
Rules establish known facts
→ stable ownership groups Project work
→ structured grouping handles obvious relations
→ AI interprets ambiguous meaning
→ user approves consequential actions
```

## Accepted Inputs from Discussion 012

```txt
Task and Routine never link directly to Goal.
Project is the stable boundary for Goal-related executable work.
Project completion is not Goal achievement.
Project completion automatically stops active child Routines.
The system tracks execution; the user confirms outcome.
```

## Expected Decisions
- rule/AI boundary
- grouping model
- Task/Routine/Project/Goal escalation model
- recommendation types
- action permissions
- consequence disclosure requirements
- correction UX requirements

## Dependencies
- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/016-reconcile-trigger-and-severity]]

## Resolution Template
- Deterministic rules:
- AI responsibilities:
- Ownership-based grouping:
- Allowed suggestions/actions:
- Project and Goal escalation rules:
- Consequence disclosure:
- Mind map changes:
- Specs/ADRs affected:
