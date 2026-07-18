# Discussion 012 — Core Product Model

## Status

Open for GPT × Claude review.

## Review Direction for Claude

Mahdi has explicitly chosen an AI-native first version of Adaptive Planner.

This discussion is not asking whether the product should be reduced to a manual Todo app in order to make the MVP cheaper. The AI-native direction is already chosen.

Please focus this review on the conceptual and technical coherence of the core product model:

- whether the concepts are distinct and necessary
- whether their ownership rules are coherent
- whether the relationships create contradictions
- whether the proposed lifecycle boundaries make sense
- whether any important core concept is missing or redundant

Do not expand this discussion into AI prompts, Reconcile behavior, database design, APIs, analytics, transaction handling, concurrency, occurrence-generation strategy, or implementation sequencing. Those topics belong to Discussions 013–022.

Related discussions:

- [[01-Open-Discussions/010-ai-first-planning-and-reconcile-roadmap]]
- [[01-Open-Discussions/011-ai-native-mvp-scope-and-mindmap-update]]

---

## 1. Scope

This discussion defines only the canonical product concepts, their boundaries, their allowed relationships, and their high-level lifecycle meaning.

It answers:

```txt
What nouns exist in the product?
What does each noun mean?
How may those nouns relate to each other?
What does completion mean at each level?
```

It does not define how those concepts are persisted, generated, scheduled, rendered, reconciled, or implemented.

---

## 2. Proposed Core Model

The initial product model contains five first-class concepts:

```txt
Goal
Project
Task
Routine
RoutineOccurrence
```

The product allows both structured and direct work:

```txt
Goal
├── zero or more Projects
│   ├── zero or more Tasks
│   └── zero or more Routines
├── zero or more direct Tasks
└── zero or more direct Routines

Standalone work:
- Project
- Task
- Routine
```

A Task or Routine may belong to:

```txt
one Goal
or one Project
or neither
```

A Task or Routine may never belong to a Goal and a Project at the same time.

This prevents artificial one-item Projects while preserving Project as the boundary for genuinely finite or independently manageable efforts.

---

## 3. Goal

### Definition

A Goal represents a desired outcome, direction, or state that matters to the user.

Examples:

- Reach English level B2
- Find a frontend job
- Improve physical fitness
- Launch a sustainable personal product

A Goal is not:

- a finite batch of work
- a list of Tasks
- a recurring behavior
- an automatically calculated percentage
- proof that a real-world outcome occurred

### Ownership

A Goal may own:

- zero or more Projects
- zero or more direct Tasks
- zero or more direct Routines

Direct Goal ownership is appropriate for work that genuinely supports the Goal but does not justify a separate Project.

Example:

```txt
Goal: Reach English level B2

Direct Task: Take a placement test
Direct Routine: Practice English for 30 minutes each day

Project: Complete English File Upper-Intermediate
- Task: Buy the course book
- Routine: Complete one lesson every weekday
```

### Evaluation Boundary

The system can observe execution, but it cannot reliably infer that the real-world Goal was achieved.

```txt
All Tasks completed
+ all Routines followed
+ all Projects completed
≠ automatically proves Goal achieved
```

Core principle:

> The system tracks execution. The user confirms outcome.

The system may truthfully report activity facts such as:

- Projects completed
- Tasks completed
- current planned work completed
- Routine adherence

It must not convert those facts into an unsupported universal Goal progress percentage.

### High-Level Lifecycle

```txt
ACTIVE → ACHIEVED | ABANDONED
```

`ACHIEVED` means the user confirms the desired outcome was reached.

`ABANDONED` means the user no longer intends to pursue the Goal.

---

## 4. Project

### Definition

A Project represents a finite or independently manageable effort.

Examples:

```txt
Goal: Reach English level B2
Project: Complete English File Upper-Intermediate

Goal: Find a frontend job
Project: Build a portfolio
Project: Prepare for interviews
Project: Apply to selected companies

Standalone Project: Move to a new apartment
```

A Project gives stable structure to one specific effort without claiming that completing it proves the parent Goal was achieved.

A Project should not be created merely to route a single direct Task or Routine to a Goal.

### Ownership

A Project may:

- belong to one Goal
- or exist standalone

A Project may contain:

- multiple Tasks
- multiple Routines

### Completion Meaning

Project completion means:

> The finite effort represented by this Project is considered complete.

It does not mean the parent Goal is achieved.

### Routine Ownership Rule

A Routine belonging to a Project is specific to that Project.

It is never shared across Projects and does not automatically become reusable elsewhere.

When a Project is completed, every active Routine belonging to it is automatically stopped.

```txt
Project ACTIVE → COMPLETED
→ active child Routines become STOPPED
```

No extra user question is required because the ownership rule already determines the outcome.

This rule applies only to Project-owned Routines. Goal-owned and standalone Routines are unaffected by Project completion.

Historical RoutineOccurrences remain part of history. The operational details of stopping future occurrences belong to later discussions.

### High-Level Lifecycle

```txt
ACTIVE → COMPLETED | STOPPED
```

`COMPLETED` means the intended effort was finished.

`STOPPED` means the effort was ended without being considered complete.

---

## 5. Task

### Definition

A Task represents one actionable, non-recurring piece of work.

Examples:

- Take a placement test
- Buy the course book
- Write the landing-page headline
- Email five selected companies

### Ownership

A Task may:

- belong directly to one Goal
- belong to one Project
- or exist standalone

A Task may not belong to both a Goal and a Project simultaneously.

If a Task belongs to a Project, any Goal context comes through that Project.

### Carry Principle

Carry is an action or transition, not a Task status.

```txt
Task remains active
→ its planned placement changes
```

The exact scheduling and history model belongs to later discussions.

### High-Level Lifecycle

```txt
ACTIVE → COMPLETED | DROPPED
```

### Explicit Exclusion

The core model does not include:

- Task-to-Task dependencies
- a generic Task `order` field

A suggested order may exist temporarily inside an AI draft, but it is not part of the canonical Task concept unless a later discussion defines one specific meaning for it.

---

## 6. Routine

### Definition

A Routine represents a recurring behavior definition.

Examples:

- Practice English for 30 minutes every weekday
- Practice interview questions three times per week
- Review product feedback every Friday

A Routine is not merely a duplicated Task. It represents repeated behavior and produces or defines individual occurrences.

### Ownership

A Routine may:

- belong directly to one Goal
- belong to one Project
- or exist standalone

A Routine may not belong to both a Goal and a Project simultaneously.

A Routine cannot belong to multiple Goals or multiple Projects.

### Ownership Meaning

A Goal-owned Routine supports the broader Goal and may remain relevant across multiple Projects.

A Project-owned Routine is specific to that Project and follows the Project lifecycle. When that Project is completed, the Routine is stopped automatically.

A standalone Routine has neither Goal nor Project ownership.

### High-Level Lifecycle

```txt
ACTIVE → STOPPED
```

Pause/resume behavior is not decided here.

---

## 7. RoutineOccurrence

### Definition

A RoutineOccurrence represents one scheduled instance of a Routine on a specific date.

The user experiences it as something like:

```txt
Today's English practice
Today's workout
Friday review
```

The technical term `RoutineOccurrence` does not need to appear in the UI.

### Ownership

Every RoutineOccurrence belongs to exactly one Routine.

### Core Behavior

A past RoutineOccurrence never becomes accumulated debt through Carry.

```txt
PENDING → DONE | MISSED
```

How occurrences are created, stored, derived, or scheduled belongs to Discussions 015 and 019.

---

## 8. Plan Is Not a Core Entity

The word “plan” may exist in the user experience, but the initial product model does not contain a persisted `Plan` entity.

The AI may produce an ephemeral proposal containing suggested Goals, Projects, Tasks, and Routines.

```txt
AI draft
→ user reviews and approves
→ approved product entities are created
```

The exact output contract belongs to Discussion 014.

---

## 9. Allowed Relationships

```txt
Goal 1 ── 0..* Project
Goal 1 ── 0..* direct Task
Goal 1 ── 0..* direct Routine
Project 1 ── 0..* Task
Project 1 ── 0..* Routine
Routine 1 ── 0..* RoutineOccurrence
```

Optional ownership:

```txt
Project may exist without Goal.
Task may exist without Goal or Project.
Routine may exist without Goal or Project.
```

Exclusive-parent rule:

```txt
Task belongs to at most one of: Goal, Project.
Routine belongs to at most one of: Goal, Project.
```

Forbidden relationships:

```txt
Task → Goal and Project simultaneously
Routine → Goal and Project simultaneously
Routine → multiple Goals
Routine → multiple Projects
Task → multiple Goals
Task → multiple Projects
Task → Task dependency
```

---

## 10. Explicitly Deferred to Later Discussions

The following are intentionally not resolved here:

- AI conversation flow and clarification questions — Discussion 013
- AI draft/output schema — Discussion 014
- Today, scheduling, occurrence generation, and execution behavior — Discussion 015
- Reconcile triggers and severity — Discussion 016
- AI Reconcile grouping and actions — Discussion 017
- guardrails, privacy, and failure handling — Discussion 018
- database schema, events, transactions, and observability — Discussion 019
- APIs, provider/runtime architecture, retries, and cost controls — Discussion 020
- validation metrics and decision thresholds — Discussion 021
- build sequence and milestone planning — Discussion 022

---

## 11. Questions for Claude

Please challenge only the core model in this discussion:

1. Are Goal, Project, Task, Routine, and RoutineOccurrence conceptually distinct enough to justify separate first-class concepts?
2. Does allowing Task and Routine to belong directly to Goal or Project remove forced wrappers without making ownership ambiguous?
3. Is the mutually exclusive parent rule conceptually sufficient?
4. Is allowing standalone Project, Task, and Routine coherent with the model?
5. Is Project-specific Routine ownership too strict, or does it correctly preserve lifecycle meaning?
6. Is automatic Routine stop on Project completion consistent with the ownership model?
7. Are the proposed high-level lifecycle meanings missing any essential terminal state?
8. Does excluding a persisted Plan entity create any conceptual gap?
9. Is any allowed or forbidden relationship still contradictory or redundant?
10. Does this model preserve the distinction between execution tracking and real-world Goal achievement without introducing unsupported progress claims?

Do not answer these questions by expanding into implementation details assigned to Discussions 013–022.

---

## 12. Expected Resolution

This discussion should close with only:

- accepted core concepts
- accepted definitions
- accepted ownership relationships
- accepted high-level lifecycle meanings
- rejected/deferred core concepts
- required changes to the current mind map
- list of formal documents that must later be updated
