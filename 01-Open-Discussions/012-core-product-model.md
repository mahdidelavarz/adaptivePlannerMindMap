# Discussion 012 — Core Product Model

## Status

Open for GPT × Claude review.

## Review Direction for Claude

Mahdi has explicitly chosen an AI-native first version of Adaptive Planner.

The purpose of this review is not to repeatedly shrink the product until AI, Projects, or Reconcile disappear in the name of keeping the MVP small. The current product direction is intentional:

```txt
Goal
→ one or more Projects
→ Tasks and Project-specific Routines
→ Today execution
→ AI-assisted Reconcile
→ user-approved adaptation
```

Claude should still identify implementation cost, architectural risk, or scope pressure when those risks create a real technical or product failure mode. However, review priority should be:

1. conceptual correctness
2. domain consistency
3. lifecycle and state-transition gaps
4. relationship and integrity problems
5. edge cases
6. AI ambiguity and unsafe inference
7. persistence and concurrency consequences that later discussions must resolve

Do not reject an agreed concept merely because removing it would make the MVP cheaper. Challenge whether the concept is technically coherent and behaviorally necessary within the chosen AI-native direction.

Related discussions:

- [[01-Open-Discussions/010-ai-first-planning-and-reconcile-roadmap]]
- [[01-Open-Discussions/011-ai-native-mvp-scope-and-mindmap-update]]

---

## 1. Purpose of This Discussion

Define the canonical product concepts, boundaries, relationships, and lifecycle semantics used by the initial version.

This discussion establishes the language of the product before the following are finalized:

- AI planning flow
- AI planning output contract
- Today and execution behavior
- AI Reconcile
- persistence model
- API contracts
- validation plan

This is a conceptual domain discussion. It should not prematurely lock database columns, API payloads, occurrence-generation algorithms, or AI-provider implementation details.

---

## 2. Agreed Core Principle

Adaptive Planner distinguishes between:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

These concepts represent different levels of user intent and execution.

The product must not flatten all work into Goals and Tasks, because a desired outcome, an organized effort, a one-time action, and a repeated behavior have different meanings and lifecycle rules.

The central hierarchy is:

```txt
Goal
└── zero or more Projects
    ├── zero or more Tasks
    └── zero or more Routines
        └── zero or more RoutineOccurrences
```

Standalone execution is also allowed:

```txt
Standalone Project
Standalone Task
Standalone Routine
```

However:

```txt
Task never links directly to Goal.
Routine never links directly to Goal.
```

A Project is always the boundary between a Goal and its executable work.

---

## 3. Goal

### 3.1 Definition

A Goal represents a desired outcome, direction, or state that matters to the user.

Examples:

- Reach English level B2
- Find a frontend job
- Improve physical fitness
- Launch a sustainable personal product

A Goal is not:

- a list of Tasks
- a finite delivery effort
- a recurring behavior
- an automatically calculated percentage
- a claim that the system can verify the user's real-world outcome

### 3.2 Goal Contains Projects, Not Direct Work

A Goal may have one or more Projects.

Example:

```txt
Goal: Find a frontend job

Project: Build a portfolio
Project: Prepare for interviews
Project: Apply to selected companies
```

Tasks and Routines related to this Goal belong to one of these Projects. They do not store a direct Goal relationship.

This prevents ambiguous ownership such as a Task being linked both to a Project and directly to a Goal.

### 3.3 Goal Outcome Cannot Be Reliably Inferred

The system cannot conclude that a Goal has been achieved only because related work was completed.

Example:

```txt
Goal: Reach English level B2

All Tasks completed
All RoutineOccurrences marked Done
All Projects completed
```

This does not prove the user has reached B2.

The system knows execution history. It does not necessarily know the real-world outcome.

Therefore:

> The system tracks execution. The user confirms outcome.

Goal achievement requires explicit user confirmation or, in a future version, an explicitly defined external verification mechanism.

The initial version does not infer Goal achievement from:

- Task completion
- Project completion
- Routine adherence
- AI confidence
- time spent
- generated plans

### 3.4 Goal Progress Language

The product must distinguish:

```txt
Work or activity completion
from
Goal outcome progress
```

The system may truthfully display facts such as:

- 2 Projects completed
- 1 Project active
- 12 of 18 current Tasks completed
- Routine adherence during the last 30 days
- all currently planned Project work has been completed

The system must not automatically translate these facts into:

```txt
Goal progress: 72%
```

unless a future explicit evaluation model makes that calculation valid.

The initial version therefore does not require a universal Goal progress percentage or Goal-type engine.

### 3.5 Goal Completion Prompt

When all active Projects under a Goal are completed or stopped, the system may ask the user to choose:

```txt
I reached this Goal
I want to create another Project for this Goal
I am no longer pursuing this Goal
```

The system must not select one automatically.

### 3.6 Proposed Goal Lifecycle

```txt
ACTIVE
→ ACHIEVED
→ or ABANDONED
```

Open lifecycle questions remain listed later in this discussion, including whether terminal states may be reopened.

---

## 4. Project

### 4.1 Definition

A Project represents a finite or independently manageable effort.

It groups the Tasks and Routines used to pursue a piece of work.

Examples:

```txt
Goal: Reach English level B2
Project: Complete English File Upper-Intermediate

Goal: Launch Adaptive Planner
Project: Build and test the first onboarding flow

Standalone Project: Move to a new apartment
```

A Project has a clearer work-completion boundary than a Goal.

A Goal may survive when one Project is completed, stopped, replaced, or found ineffective.

### 4.2 Project May Be Standalone

A Project may exist without a Goal.

This avoids forcing users to invent artificial Goals for finite work such as:

- move to a new apartment
- prepare tax documents
- redesign a personal website
- organize an event

Conceptually:

```txt
Project.goalId is optional.
```

The exact storage field belongs in the data-model discussion.

### 4.3 Project Owns Its Tasks and Routines

A Task or Routine may belong to at most one Project.

A Project may contain multiple Tasks and multiple Routines.

Project-specific work must not be shared across Projects.

In particular:

> A Routine belongs to one Project or is standalone. It is never shared across multiple Projects.

This is an intentional product rule, not merely a database limitation.

If two Projects require similar recurring behavior, they use separate Routines so each can have an independent lifecycle and execution history.

### 4.4 Project Completion

A Project may be marked completed when its defined effort is finished.

Project completion means:

```txt
The user considers this finite effort complete.
```

It does not mean its parent Goal has been achieved.

### 4.5 Automatic Routine Stop on Project Completion

When a Project transitions to `COMPLETED`, every active Routine belonging to that Project is automatically transitioned to `STOPPED`.

This behavior is mandatory and does not require another user confirmation.

Reason:

- Project Routines are defined specifically for that Project.
- They are not intended to become reusable or cross-Project habits automatically.
- Continuing them after Project completion would create orphaned behavior with misleading context.
- Asking the user what to do with each Routine adds unnecessary decision work when the ownership rule already determines the answer.

The lifecycle rule is:

```txt
Project ACTIVE → COMPLETED
→ all active child Routines become STOPPED
→ no future occurrences are scheduled for those Routines
```

Historical RoutineOccurrences remain part of execution history.

The handling of already-generated future occurrences, pending occurrences on the completion date, transactions, and failure recovery belongs in Discussions 015 and 019.

### 4.6 Project Stop versus Complete

A distinction is currently proposed between:

```txt
COMPLETED
The intended finite effort was finished.

STOPPED
The user ended the effort without considering it completed.
```

A later decision must clarify whether stopping a Project also automatically stops all of its active Routines. The likely invariant is yes, because Project-specific Routines cannot remain active without their Project, but this must be reviewed explicitly rather than silently assumed.

### 4.7 Project Work Completion

The system may calculate descriptive work-completion information at Project level, such as:

```txt
completed Tasks / current non-dropped Tasks
```

This is only a summary of defined work.

It is not:

- proof of Project success in the real world
- Goal progress
- an AI-generated probability of success

Whether a percentage should be displayed, and how Task additions or dropped Tasks affect it, remain open UX and data questions.

### 4.8 Proposed Project Lifecycle

```txt
ACTIVE
→ COMPLETED
→ or STOPPED
```

No automatic transition from Project completion to Goal achievement exists.

---

## 5. Task

### 5.1 Definition

A Task represents one actionable, non-recurring piece of work.

Examples:

- Buy the course book
- Write the landing-page headline
- Update the resume introduction
- Email five selected companies

### 5.2 Task Ownership

A Task may:

- belong to exactly one Project
- or exist standalone

A Task never links directly to a Goal.

Conceptually:

```txt
Task.projectId is optional.
Task.goalId does not exist.
```

### 5.3 Carry Is a Transition, Not a Status

`CARRIED` is not a persisted Task status.

Carry means:

```txt
Task remains ACTIVE
planned date changes
carry history or count changes
```

This avoids contradictory states such as a Task being both active and carried.

The exact scheduling fields and event history belong in later discussions.

### 5.4 Proposed Task Lifecycle

```txt
ACTIVE
→ COMPLETED
→ or DROPPED
```

Whether Tasks may be reopened is still open.

### 5.5 Task Ordering and Dependencies

The MVP does not currently accept:

- a general Task dependency graph
- a generic persisted `order` field

A generic `order` field is rejected for now because its meaning is ambiguous:

- UI display order
- required execution sequence
- priority
- AI suggestion order
- Project ordering
- Today ordering

AI may return a suggested sequence inside an ephemeral draft, but that does not automatically become a core Task property.

---

## 6. Routine

### 6.1 Definition

A Routine represents a recurring behavior definition.

Examples:

- Study for 30 minutes every weekday
- Practice interview questions three times per week
- Review product feedback every Friday

A Routine is not a repeated Task copy. It owns recurrence behavior and generates or derives individual RoutineOccurrences.

### 6.2 Routine Ownership

A Routine may:

- belong to exactly one Project
- or exist standalone

A Routine never links directly to a Goal.

Conceptually:

```txt
Routine.projectId is optional.
Routine.goalId does not exist.
```

A Routine cannot belong to multiple Projects.

A Project Routine is specific to that Project and follows the Project lifecycle.

### 6.3 Project Completion Rule

When the owning Project is completed, the Routine is stopped automatically.

The user is not asked whether it should:

- continue standalone
- move to another Project
- remain active

Those actions are deliberately excluded from the automatic completion flow.

If the user wants a similar Routine elsewhere, they create or approve a new Routine under the relevant Project or as standalone work.

This preserves clean ownership and prevents historical identity from being silently repurposed.

### 6.4 Routine Occurrences Do Not Carry

A past RoutineOccurrence does not become accumulated future debt.

```txt
Past occurrence
→ DONE or MISSED
→ never CARRY
```

The Routine definition may continue producing future occurrences while active.

### 6.5 Proposed Routine Lifecycle

```txt
ACTIVE
→ STOPPED
```

Pause and resume are deferred unless later execution UX demonstrates they are necessary.

---

## 7. RoutineOccurrence

### 7.1 Definition

A RoutineOccurrence represents one scheduled instance of one Routine on one local date.

Example:

```txt
Routine: Study English every weekday
Occurrence: Study English on 2026-07-20
```

### 7.2 Ownership

Every RoutineOccurrence belongs to exactly one Routine.

It cannot exist independently, belong directly to a Project, or link directly to a Goal.

### 7.3 Proposed Lifecycle

```txt
PENDING
→ DONE
→ or MISSED
```

`SKIPPED` is not accepted automatically. It should only exist if a later UX discussion defines a real user action and semantics for it.

### 7.4 Visibility

The user perceives RoutineOccurrences as daily or scheduled instances of a Routine.

The product does not need to expose the technical term `RoutineOccurrence` in the interface.

### 7.5 Materialization Is Deferred

This discussion does not decide whether occurrences are:

- pre-generated
- generated lazily
- derived dynamically
- persisted only after interaction

That decision belongs in Discussions 015 and 019.

---

## 8. Plan

### 8.1 No Persisted Plan Entity

The initial version does not include a persisted `Plan` entity.

The AI-generated plan is an ephemeral proposal containing suggested product concepts.

```txt
AI generates a draft
→ user edits, accepts, or rejects items
→ only approved entities are persisted
```

Depending on the resolved output contract, approved items may include:

- Goal
- Project
- Task
- Routine

The word “plan” may still be used in the UX as a human-readable description of the current organized work.

The existence of the word in the UI does not require a `plans` table or a separate domain aggregate.

---

## 9. Canonical Relationship Rules

The current proposed conceptual relationships are:

```txt
Goal 1 ── 0..* Project
Project 0..1 ── 0..* Task
Project 0..1 ── 0..* Routine
Routine 1 ── 0..* RoutineOccurrence
```

Expressed as ownership rules:

```txt
Project may belong to one Goal or be standalone.
Task may belong to one Project or be standalone.
Routine may belong to one Project or be standalone.
RoutineOccurrence must belong to one Routine.
```

Forbidden direct relationships:

```txt
Task → Goal
Routine → Goal
RoutineOccurrence → Project
RoutineOccurrence → Goal
Task → Routine
Routine → multiple Projects
Task → multiple Projects
```

No general Project hierarchy, nested Project tree, Milestone entity, Path entity, or Task dependency graph is accepted for the MVP.

---

## 10. Example Models

### 10.1 Goal With Multiple Projects

```txt
Goal: Find a frontend job

Project: Build portfolio
- Task: Select three projects
- Task: Write case studies
- Routine: Work on portfolio every weekday

Project: Prepare for interviews
- Task: Create question list
- Routine: Practice technical questions three times per week

Project: Apply to companies
- Task: Create target-company list
- Routine: Submit two focused applications each weekday
```

Completing or stopping one Project does not complete or abandon the Goal.

### 10.2 Goal Where Project Work Finishes but Outcome Is Unknown

```txt
Goal: Reach English level B2

Project: Complete Upper-Intermediate course
- all Tasks completed
- child Routines automatically stopped
- Project marked COMPLETED
```

The system may say:

```txt
The current Project is complete.
All active work currently defined for this Goal is complete.
```

It must not say:

```txt
You have reached B2.
```

The user decides whether to:

- mark the Goal achieved
- create another Project
- abandon the Goal

### 10.3 Standalone Project

```txt
Project: Move to a new apartment
- Task: Book a moving company
- Task: Pack the kitchen
- Routine: Pack for 30 minutes each evening until moving day
```

No Goal is required.

When the Project is completed, its packing Routine is stopped automatically.

### 10.4 Standalone Task and Routine

```txt
Task: Buy groceries
Routine: Drink water every day
```

Neither requires a Project or Goal.

---

## 11. Lifecycle Summary

Current proposed states:

```txt
Goal
ACTIVE → ACHIEVED | ABANDONED

Project
ACTIVE → COMPLETED | STOPPED

Task
ACTIVE → COMPLETED | DROPPED

Routine
ACTIVE → STOPPED

RoutineOccurrence
PENDING → DONE | MISSED
```

Mandatory cascade currently accepted:

```txt
Project → COMPLETED
causes
all active child Routines → STOPPED
```

The technical implementation must preserve historical occurrences and apply the cascade consistently.

---

## 12. Accepted Direction

The following direction is currently agreed by Mahdi and should be treated as the product-owner decision under review:

- The initial version is AI-native.
- Goal, Project, Task, Routine, and RoutineOccurrence are distinct first-class concepts.
- A Goal has zero or more Projects.
- A Project may belong to a Goal or exist standalone.
- Tasks and Routines belong to one Project or exist standalone.
- Tasks and Routines never link directly to Goal.
- A Routine never belongs to multiple Projects.
- Project-specific Routines stop automatically when the Project is completed.
- The system tracks execution and work completion, not verified real-world Goal achievement.
- Goal achievement requires explicit user confirmation.
- Project completion does not imply Goal achievement.
- No persisted Plan entity exists in the MVP.
- Carry is a Task transition, not a Task status.
- RoutineOccurrences never Carry.
- A generic Task `order` field and dependency graph are not accepted.

---

## 13. Questions Still Open for Technical Review

Claude should focus especially on the following unresolved technical and semantic questions.

### Goal

1. Can an `ACHIEVED` or `ABANDONED` Goal be reopened?
2. Can a Goal be marked achieved while active Projects still exist?
3. If the Goal is abandoned, what happens to active Projects and their child work?
4. Is Goal deletion allowed, or only terminal status changes?
5. When all Projects are stopped or completed, should the system always prompt for a Goal decision or only during Reconcile?

### Project

6. Does Project `STOPPED` apply the same automatic Routine-stop cascade as `COMPLETED`?
7. What happens to active child Tasks when a Project is completed?
8. Must all child Tasks be completed or dropped before a Project can be marked completed?
9. Can the user force-complete a Project with unresolved Tasks, and if so, what status do those Tasks receive?
10. Can a completed or stopped Project be reopened?
11. Can a Project move between Goals?
12. Can a standalone Project later be attached to a Goal?
13. Can a Goal-owned Project later become standalone?
14. Should Project work completion be displayed as a count, percentage, or both?
15. How should dropped Tasks affect Project work-completion summaries?

### Task

16. Can a Task move between Projects or become standalone?
17. Can a standalone Task later be attached to a Project?
18. Are Task terminal states reopenable?
19. What is the exact behavior of unresolved Tasks when their Project stops or completes?
20. Is Task priority part of this conceptual model or deferred?

### Routine

21. Does Project `STOPPED` automatically stop child Routines, as Project `COMPLETED` does?
22. Can a stopped Routine be restarted under the same Project?
23. Can a Project Routine ever be detached and made standalone manually, or is that forbidden entirely?
24. If a user needs the same behavior in a new Project, should the product duplicate the Routine or create a fresh one with no historical link?
25. What happens to a RoutineOccurrence scheduled for the same date on which its Project is completed?
26. What happens to already-materialized future RoutineOccurrences when the Routine is automatically stopped?

### Relationship Integrity

27. What transaction boundary guarantees that Project completion and Routine stopping cannot diverge?
28. How should concurrent Project completion and Routine completion requests be resolved?
29. Which relationship changes require historical events for auditability and AI context?
30. Are ownership changes allowed after execution history exists, or should they be heavily constrained?

### AI Semantics

31. May Planning AI propose standalone Tasks or Routines, or must all AI-created work belong to a Project?
32. When the user gives a Goal, must Planning AI always propose at least one Project before proposing Tasks or Routines?
33. How should AI distinguish a Goal from a standalone Project in ambiguous natural language?
34. During Reconcile, can AI recommend stopping or replacing a Project?
35. What level of confirmation is required before AI actions complete a Project and automatically stop its Routines?
36. How should AI explain the difference between completed Project work and unverified Goal achievement?

### User Visibility

37. Which terms are explicitly shown in the interface: Goal, Project, Task, Routine?
38. Should users see work-completion summaries at Goal level, Project level, or both?
39. What wording prevents users from interpreting activity completion as verified outcome progress?
40. How should the interface explain that Project completion automatically stops its Routines?

---

## 14. Out of Scope for This Discussion

The following remain for later ordered discussions:

- exact AI conversation and clarification flow
- Planning AI JSON schema
- recurrence calculation algorithm
- RoutineOccurrence materialization strategy
- Today composition
- Reconcile trigger and severity
- semantic grouping and AI recommendations
- database table and column design
- cascade implementation and transaction mechanics
- API endpoints
- prompt versioning
- AI provider choice
- validation thresholds
- milestone and delivery plan

---

## 15. Mind Map Impact

If this direction is accepted, update the current mind map as follows.

### Product Vision

Clarify:

```txt
The system tracks execution and adaptation.
The user confirms whether the real-world Goal was achieved.
```

### MVP Core Loop

Use:

```txt
Goal or standalone intention
→ AI proposes Project structure
→ AI proposes Tasks and Project-specific Routines
→ user approves
→ execution in Today
→ AI Reconcile at Task, Routine, Project, and Goal levels
```

### User Flow

Add:

- Goal creation with one or more proposed Projects
- standalone Project creation
- standalone Task and Routine creation
- Project completion flow
- automatic child-Routine stopping
- Goal review when no active Project work remains

### AI Responsibilities

Planning AI must distinguish:

- desired outcome → Goal
- finite organized effort → Project
- single action → Task
- recurring project-specific behavior → Routine

Reconcile AI may surface problems at:

- Task level
- Routine level
- Project level
- Goal level

### AI Guardrails

Add:

- AI must not infer Goal achievement from execution completion.
- AI must not silently mark a Goal achieved.
- AI must explain that Project completion stops Project Routines.
- Completing a Project is consequential because of the Routine cascade and requires user approval.

### Data Events

Later event design must cover:

- Project created
- Project completed
- Project stopped
- Project relationship changed
- Routine automatically stopped due to Project completion
- Goal marked achieved by user
- Goal abandoned by user

### Current Decisions

Record all items from Section 12.

### Open Questions

Use Section 13 until resolved.

---

## 16. Formal Documents Affected

Resolution of this discussion will require explicit updates to at least:

- `day-0-onboarding.md`
- `reconcile-ux.md`
- `data-model-phase-1.md`
- `api-contracts-phase-1.md`
- `validation-plan.md`
- `metrics.md`
- `phase-1-plan.md`
- relevant ADRs that previously excluded Project, Routine, or AI runtime

No existing ADR or specification is silently overridden by this discussion.

---

## 17. Resolution Template

When closing this discussion, record:

```txt
Accepted concepts:
Accepted relationship invariants:
Accepted lifecycle states:
Accepted cascade rules:
Rejected or deferred concepts:
Remaining implementation questions moved to later discussions:
Mind map changes:
Formal documents affected:
```
