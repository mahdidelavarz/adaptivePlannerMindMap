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

The main hierarchy is:

```txt
Goal
└── zero or more Projects
    ├── zero or more Tasks
    └── zero or more Routines
        └── zero or more RoutineOccurrences
```

Standalone work is also allowed:

```txt
Standalone Project
Standalone Task
Standalone Routine
```

Core ownership rule:

```txt
Task never links directly to Goal.
Routine never links directly to Goal.
```

A Project is the boundary between a Goal and its executable work.

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

A Goal may contain zero or more Projects.

Tasks and Routines do not belong directly to a Goal. They belong to a Project or exist standalone.

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

- Buy the course book
- Write the landing-page headline
- Update the resume introduction
- Email five selected companies

### Ownership

A Task may:

- belong to one Project
- or exist standalone

A Task never links directly to a Goal.

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

- Study for 30 minutes every weekday
- Practice interview questions three times per week
- Review product feedback every Friday

A Routine is not merely a duplicated Task. It represents repeated behavior and produces or defines individual occurrences.

### Ownership

A Routine may:

- belong to one Project
- or exist standalone

A Routine never links directly to a Goal.

A Routine cannot belong to multiple Projects.

A Project-specific Routine follows the lifecycle of its Project. When that Project is completed, the Routine is stopped automatically.

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
Project 1 ── 0..* Task
Project 1 ── 0..* Routine
Routine 1 ── 0..* RoutineOccurrence
```

Optional ownership:

```txt
Project may exist without Goal.
Task may exist without Project.
Routine may exist without Project.
```

Forbidden relationships:

```txt
Task → Goal
Routine → Goal
Routine → multiple Projects
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
2. Does requiring Project as the only bridge between Goal and executable work create any conceptual contradiction?
3. Is allowing standalone Project, Task, and Routine coherent with the hierarchy?
4. Is Project-specific Routine ownership too strict, or does it correctly preserve lifecycle meaning?
5. Is automatic Routine stop on Project completion consistent with the proposed ownership model?
6. Are the proposed high-level lifecycle meanings missing any essential terminal state?
7. Does excluding a persisted Plan entity create any conceptual gap?
8. Is any relationship listed as allowed actually ambiguous or redundant?
9. Is any forbidden relationship necessary for the chosen product direction?
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
