# Discussion 013 — AI Planning Entry and Conversation Flow

## Status

Open for GPT × Claude review.

## Review Direction for Claude

Mahdi has already chosen an AI-native first version of Adaptive Planner.

This discussion is not asking whether AI-assisted planning should be removed, postponed, or replaced with a manual Todo-only MVP. The AI-native direction is already accepted.

Please review only the coherence of the proposed entry and conversation flow:

- contradictions with Discussion 012
- missing conversation states
- ambiguous transitions
- edge cases that leave the user trapped or uncertain
- conflicts with earlier accepted decisions
- the smallest coherent fixes required

Do not redesign the product from zero, reject accepted decisions merely to reduce MVP size, or expand this review into the final AI output schema, prompt engineering, database design, provider architecture, Reconcile, analytics, or implementation sequencing.

Related discussions:

- [[01-Open-Discussions/010-ai-first-planning-and-reconcile-roadmap]]
- [[01-Open-Discussions/011-ai-native-mvp-scope-and-mindmap-update]]
- [[01-Open-Discussions/012-core-product-model]]

---

## 1. Scope

This discussion defines how a user starts AI-assisted planning and moves from an initial intention to a reviewable draft.

It answers:

```txt
Where may the planning flow begin?
What may the user provide first?
When should the AI ask a question?
When should it stop asking and produce a draft?
How may the user leave, restart, or recover?
```

It does not define:

- the final AI JSON contract — Discussion 014
- Task, Routine, Today, or execution behavior — Discussion 015
- Reconcile — Discussions 016 and 017
- safety, privacy, and detailed model/provider failure policy — Discussion 018
- database schema, APIs, retries, cost controls, or implementation — Discussions 019 and 020

---

## 2. Proposed Entry Model

The AI planning flow supports both broad and narrow entry intentions.

A user may begin with:

```txt
1. a desired outcome or direction
2. a finite effort or project idea
3. one or more Tasks
4. one or more Routines
5. an unstructured free-form intention
```

Examples:

```txt
I want to find a frontend job.
Help me prepare for IELTS.
I need to launch my portfolio.
Remind me to practice English every weekday.
I need to email three companies tomorrow.
My life is disorganized and I do not know where to start.
```

The user is not required to create or name a Goal before entering the flow.

The AI interprets the intention using the accepted product concepts from Discussion 012:

```txt
Goal
Project
Task
Routine
```

`RoutineOccurrence` is not directly created through this conversation. It belongs to later scheduling and execution behavior.

The AI may propose no Goal at all when the user's request only justifies a standalone Project, Task, or Routine.

The AI must not force every request into a Goal hierarchy.

---

## 3. Entry Points

AI-assisted planning is available from three product entry points:

### 3.1 Global AI Planning Entry

A primary action such as:

```txt
Plan with AI
```

This opens a new planning conversation without requiring existing context.

### 3.2 Contextual Entry from an Existing Goal or Project

From an existing Goal or Project, the user may choose an action such as:

```txt
Plan next steps with AI
```

The selected Goal or Project becomes explicit conversation context.

The AI must not silently move proposed items outside that context unless it tells the user why and asks for confirmation in the draft review stage.

### 3.3 Manual Creation Entry

Manual creation remains available outside the AI flow for:

- Goal
- Project
- Task
- Routine

Manual creation is a parallel path, not a failure-only fallback.

The user must never be forced to use AI to create or edit canonical product entities.

---

## 4. First Screen and Empty State

The first screen uses one open input rather than a mandatory Goal form.

Proposed primary prompt:

> What do you want to make progress on?

Supporting text:

> Describe an outcome, project, task, routine, or even a rough intention. I’ll help turn it into a plan you can review before anything is created.

Suggested examples may be shown as optional starters:

- Find a better frontend job
- Build and publish my portfolio
- Practice English every weekday
- Organize the tasks I need to finish this week

The UI must make two facts clear before the user submits:

```txt
The AI may ask a small number of questions.
Nothing is created until the user reviews and approves the draft.
```

---

## 5. Conversation Style

The interaction is hybrid:

```txt
conversational input
+ structured quick-reply options when useful
+ free-text escape at every clarification step
```

The AI may use buttons, chips, date pickers, or short option lists for bounded questions such as:

- target date
- preferred frequency
- available days
- ownership under an existing Goal or Project

The user is never limited to predefined options when those options do not fit.

The flow must not become a long form disguised as a chat.

The system should prefer one question at a time unless two tightly coupled fields are easier to answer together, such as:

```txt
Which days, and roughly how long each time?
```

---

## 6. Conversation States

The proposed high-level conversation state model is:

```txt
EMPTY
→ INTERPRETING
→ CLARIFYING
→ DRAFT_READY
→ REVIEWING
→ APPROVED | CANCELLED
```

Recovery states:

```txt
INTERPRETING | CLARIFYING
→ INPUT_BLOCKED
→ CLARIFYING | DRAFT_READY | CANCELLED

INTERPRETING | CLARIFYING | DRAFT_READY
→ GENERATION_FAILED
→ RETRYING | MANUAL_FALLBACK | CANCELLED
```

State meanings:

### EMPTY

No meaningful user intention has been submitted.

### INTERPRETING

The system is identifying likely Goals, Projects, Tasks, Routines, ownership, and missing information.

### CLARIFYING

The system asks only questions required to produce a useful and reviewable draft.

### INPUT_BLOCKED

The system cannot proceed safely or coherently because the input is too vague, contradictory, unsupported, or missing a required planning constraint.

### DRAFT_READY

A draft has been generated but no canonical product entity has been created.

### REVIEWING

The user is editing, removing, accepting, or rejecting proposed items.

Detailed draft editing and approval behavior belongs to Discussion 014.

### APPROVED

The user has explicitly approved creation of the accepted draft content.

### CANCELLED

The user exits without creating the proposed entities.

### GENERATION_FAILED

The draft could not be produced because of a model, provider, network, parsing, or internal failure.

Detailed retry and provider behavior belongs to Discussions 018 and 020.

---

## 7. Clarification Policy

Clarification is selective, not mandatory for every request.

The AI should generate a draft immediately when the request is already specific enough to produce a useful proposal without inventing important facts.

Example:

```txt
User: Create a routine to practice English for 30 minutes every weekday.

Expected behavior:
Generate a draft directly.
Do not ask what the user's life purpose is, because the software is a planner, not an overeager philosophy student.
```

The AI asks a clarification only when the answer materially changes one or more of:

- whether the intention is a Goal, Project, Task, or Routine
- ownership under an existing Goal or Project
- the actionable content of the draft
- timing or recurrence required to make the item meaningful
- user constraints that prevent an unrealistic or contradictory proposal

---

## 8. Mandatory, Optional, and Forbidden Questions

### 8.1 Mandatory Clarification

A question is mandatory only when the AI cannot generate a coherent draft without making a high-impact assumption.

Typical mandatory cases:

1. A recurring behavior is requested but no usable recurrence can be inferred.
2. A date-sensitive request contains conflicting dates or impossible timing.
3. The user references an existing Goal or Project ambiguously and the intended context cannot be identified.
4. The input contains two materially different intentions and the system cannot determine whether they should be one plan or separate drafts.
5. The requested draft depends on a hard constraint the user explicitly mentions but does not define.

### 8.2 Optional Clarification

Optional questions improve the draft but are not required to produce one.

Examples:

- preferred intensity
- available time per day
- preferred working days
- desired target date
- whether a standalone item should later be connected to a Goal

The AI may ask an optional question only when its expected value is high enough to justify another interaction step.

### 8.3 Forbidden Clarification

The AI must not ask for:

- sensitive personal data not required for planning
- a complete life history
- psychological diagnosis
- motivation scoring questionnaires before planning
- information already provided in the conversation or known product context
- implementation details the user should not need to understand
- confirmation of obvious low-impact assumptions that can be shown in the draft instead

The AI must not repeatedly ask differently worded versions of the same unresolved question.

---

## 9. Clarification Limit

The default clarification budget is:

```txt
maximum 3 AI clarification turns before producing a draft
```

A clarification turn may contain one question or one tightly coupled question group.

The AI should usually need zero to two turns.

After the third clarification turn, it must do one of the following:

```txt
1. generate a draft with clearly labeled assumptions
2. generate a smaller partial draft for the understood portion
3. explain the single blocking ambiguity and offer manual creation or restart
```

It must not continue an open-ended interview.

The limit resets only when:

- the user materially changes the original intention
- the user explicitly restarts the flow
- the user approves splitting the request into separate planning flows

---

## 10. Skip Clarification

At any clarification step, the user may choose:

```txt
Draft now
```

When this happens, the AI generates the best coherent draft it can using:

- explicit user input
- existing selected Goal or Project context
- low-risk assumptions

Every material assumption must be visible in the reviewable draft.

The AI must not silently invent:

- deadlines
- recurrence schedules
- ownership relationships
- measurable Goal outcomes
- user availability

If one of those is essential and cannot be safely assumed, the AI may produce a partial draft rather than fabricating a complete one.

---

## 11. Direct Task-Only and Routine-Only Requests

The user may directly request:

```txt
only a Task
only multiple Tasks
only a Routine
only multiple Routines
only a Project
```

The AI must not create a Goal or Project merely to make the output look more structured.

Examples:

```txt
Email the landlord tomorrow.
→ standalone Task draft

Practice English every weekday.
→ standalone Routine draft

Plan the steps needed to move apartments.
→ standalone Project with Tasks, if the finite effort justifies a Project
```

The accepted ownership rules from Discussion 012 remain unchanged:

```txt
Task or Routine may belong to one Goal, one Project, or neither.
Task or Routine may not belong to Goal and Project simultaneously.
```

---

## 12. Vague, Contradictory, or Incomplete Input

### 12.1 Vague but Usable

Example:

```txt
I want to get better at English.
```

The AI should ask one high-value question or generate a conservative draft with assumptions, depending on how much structure can be usefully inferred.

### 12.2 Too Vague to Plan

Example:

```txt
Fix my life.
```

The AI should narrow the entry point without pretending it understands the entire problem.

Proposed response shape:

```txt
Which area feels most important to improve first?
- work or study
- health
- personal organization
- relationships
- something else
```

A free-text answer remains available.

### 12.3 Contradictory Input

Example:

```txt
Create a daily routine, but I only want to do it once.
```

The AI must identify the contradiction and ask the smallest question that resolves it.

It must not silently choose one side.

### 12.4 Incomplete but Non-Blocking

If missing information can be represented as an editable assumption, the AI should produce the draft rather than prolonging clarification.

---

## 13. Pilot Domain Boundary

The pilot is domain-light rather than restricted to a tiny topic allowlist.

The AI may assist with ordinary personal planning across domains such as:

- work and career
- learning and study
- personal projects
- home and life administration
- fitness and general wellbeing habits
- creative work
- job search
- product building

The product does not claim expert authority in regulated or high-stakes domains.

For planning requests involving medical treatment, mental-health crisis, legal strategy, financial investment, physical danger, or other high-stakes decisions:

```txt
The AI may help organize user-provided actions or questions.
It must not replace qualified professional judgment.
It must not invent expert instructions.
```

Detailed safety language and refusal boundaries belong to Discussion 018.

Therefore, Discussion 013 does not define a brittle fixed topic allowlist. It defines a planning capability boundary:

> The system may structure ordinary intentions into planning entities, but it must not act as an unqualified domain expert.

---

## 14. Out-of-Scope Requests

When the user asks for something outside planning capability, the system should:

1. state the boundary briefly
2. preserve any planning-relevant portion
3. offer the nearest supported planning action

Example:

```txt
User: Diagnose my symptoms and create a treatment plan.

Supported response direction:
I cannot diagnose or prescribe treatment. I can help you prepare questions for a clinician, track appointments, or organize actions already recommended by a qualified professional.
```

The system must not create misleading Tasks or Routines that imply unsupported professional advice.

---

## 15. Exit, Restart, and Context Change

The user may leave the flow at any point before approval.

Available actions:

```txt
Cancel
Restart
Edit original intention
Draft now
Continue manually
```

### Cancel

Ends the flow without creating canonical entities.

### Restart

Clears the current planning conversation and returns to the empty state.

### Edit Original Intention

Returns to the initial input while preserving nothing unless the user explicitly chooses to keep the current conversation.

### Continue Manually

Leaves the AI flow and opens the relevant manual creation path.

### Context Change

If the user materially changes the intention during clarification, the AI must acknowledge the change and either:

```txt
continue as one revised flow
or propose splitting it into separate drafts
```

It must not quietly blend unrelated intentions into one hierarchy.

---

## 16. Session Recovery

If the user leaves the screen before approval, the product may preserve an unapproved conversation draft for short-term recovery.

However:

- no Goal, Project, Task, or Routine is created before explicit approval
- recovery must clearly show that the content is still an unapproved draft
- the user may discard and restart
- detailed persistence duration and storage behavior belong to Discussions 018–020

The conversation must never reappear as if it were an accepted plan.

---

## 17. Failure Behavior

When interpretation or generation fails, the user should see a recoverable state rather than a dead end.

The flow offers:

```txt
Retry
Edit input
Continue manually
Cancel
```

If partial user input remains available, it should not be lost merely because generation failed.

The product must not claim that entities were created unless creation actually succeeded after explicit approval.

Detailed error taxonomy, retry limits, idempotency, and provider fallback belong to Discussions 018–020.

---

## 18. Proposed End-to-End Flow

```txt
User opens AI planning
→ enters a Goal, Project, Task, Routine, or rough intention
→ system interprets likely structure

If already specific enough:
→ generate draft

If a high-impact ambiguity exists:
→ ask up to 3 clarification turns
→ user answers, chooses Draft now, restarts, cancels, or continues manually

Then:
→ generate reviewable draft
→ user reviews proposed Goal / Project / Task / Routine structure
→ user edits, removes, approves, or cancels

Only after explicit approval:
→ canonical entities may be created
```

Core rule:

> Conversation produces a proposal. Approval creates product entities.

---

## 19. Proposed Decisions

### Accepted Entry Model

- The flow accepts a Goal, Project idea, Task, Routine, or free-form intention.
- A Goal is not required.
- Narrow requests may produce standalone entities.
- Entry is available globally and contextually from an existing Goal or Project.
- Manual creation remains permanently available as a parallel path.

### Clarification Policy

- Clarification is selective.
- The AI asks only questions that materially affect the draft.
- Maximum default clarification budget is three turns.
- The user may request `Draft now` at any point.
- Missing non-blocking details become visible draft assumptions.
- High-impact facts must not be silently invented.

### Interaction Model

- The flow is conversational with structured quick replies where useful.
- Free text remains available at every step.
- The system prefers one question at a time.
- The conversation must not become an exhaustive intake form.

### Domain Boundary

- The pilot supports ordinary personal planning across broad life domains.
- There is no brittle fixed topic allowlist in this discussion.
- The system structures plans but does not replace professional judgment in high-stakes domains.

### Exit and Failure Behavior

- The user may cancel, restart, edit the original intention, draft immediately, or continue manually.
- No canonical entities are created before explicit approval.
- Generation failure preserves user input and offers retry, edit, manual fallback, or cancel.

---

## 20. Mind Map Impact — Record Only, Do Not Apply Yet

After acceptance, the Mind Map should later add or revise:

### AI Responsibilities

- interpret broad or narrow planning intent
- ask only high-value clarification questions
- respect the three-turn clarification budget
- generate reviewable drafts with visible assumptions
- preserve manual creation as a parallel path
- never create canonical entities before approval

### User Flow

Add:

```txt
Global AI entry
Contextual Goal/Project entry
Manual creation path
Clarification loop
Draft now path
Draft review boundary
Cancel/restart/manual fallback
Generation failure recovery
```

### AI Guardrails

Add:

- no forced Goal hierarchy
- no silent high-impact assumptions
- no repeated questioning for already-known information
- no expert substitution in high-stakes domains
- no claim of creation before approval succeeds

### Open Questions to Remove or Narrow

- whether the user must start with a Goal
- whether direct Task-only or Routine-only requests are supported
- whether the experience is chat or form
- whether manual creation exists
- whether clarification may be skipped

---

## 21. Documents Affected — Record Only, Do Not Update Yet

After Discussion 021 consolidation, accepted decisions from this discussion should update or create:

- AI planning conversation flow specification
- product UX flow documentation
- AI responsibility and guardrail specification
- draft review specification in coordination with Discussion 014
- failure and recovery specification in coordination with Discussion 018
- runtime/API specification in coordination with Discussion 020
- analytics event specification in coordination with Discussion 019
- implementation plan in Discussion 022

Potential ADR impact:

- an ADR may be required for the AI planning entry model and approval boundary if no existing ADR already locks them

No Mind Map or formal document should be updated from this discussion alone before consolidation after Discussion 021.

---

## 22. Structured Review Request for Claude

Review the proposal above under the following constraints.

### Review Objective

Find only:

1. contradictions with Discussion 012 or earlier accepted decisions
2. missing conversation states or transitions
3. ambiguous ownership or context behavior
4. edge cases that could trap the user, lose input, or create entities unexpectedly
5. conflicts between the clarification limit and the requirement for coherent drafts
6. domain-boundary wording that is internally inconsistent
7. places where responsibility leaks into Discussions 014–020
8. the smallest coherent fixes required

### Do Not

- redesign the product from zero
- reject AI-native planning because a manual MVP would be cheaper
- reject a decision merely to reduce MVP scope
- invent the final AI JSON contract
- define database tables, APIs, provider selection, token budgets, retry algorithms, or analytics schemas
- expand into Reconcile
- replace the accepted core concepts from Discussion 012

### Required Response Format

For each finding, use:

```txt
Finding ID:
Severity: Blocking | Important | Minor
Location:
Problem:
Why it conflicts or fails:
Smallest coherent fix:
Affected earlier decision or later discussion:
```

Then finish with:

```txt
A. Blocking issues that must be resolved before acceptance
B. Important clarifications that improve coherence
C. Minor wording or boundary fixes
D. Confirmed decisions that are coherent as written
E. Minimal revised decision list
F. Mind Map impact changes, if any
G. Affected formal documents, if any
```

Do not provide a replacement product design. Preserve all coherent decisions and change only what is necessary.