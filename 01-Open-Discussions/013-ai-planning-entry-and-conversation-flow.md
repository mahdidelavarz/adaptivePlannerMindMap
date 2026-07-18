# Discussion 013 — AI Planning Entry and Conversation Flow

## Status

Accepted after GPT × Claude review.

Claude review produced no blocking findings. Four minimal coherence fixes were accepted:

1. add the direct `INTERPRETING → DRAFT_READY` transition
2. map out-of-scope responses to an explicit conversation state behavior
3. define what happens when a new global entry collides with an active unapproved draft
4. define `RETRYING` and `MANUAL_FALLBACK`

Related discussions:

- [[01-Open-Discussions/010-ai-first-planning-and-reconcile-roadmap]]
- [[01-Open-Discussions/011-ai-native-mvp-scope-and-mindmap-update]]
- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/018-ai-safety-privacy-and-failure-policy]]

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
- detailed safety, privacy, and provider-failure policy — Discussion 018
- analytics, database schema, APIs, cost controls, or implementation — Discussions 019 and 020

---

## 2. Accepted Entry Model

The AI planning flow supports both broad and narrow intentions.

A user may begin with:

```txt
1. a desired outcome or direction
2. a finite effort or Project idea
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

The AI interprets the intention using the accepted concepts from Discussion 012:

```txt
Goal
Project
Task
Routine
```

`RoutineOccurrence` is not directly created through this conversation. It belongs to later scheduling and execution behavior.

The AI may propose no Goal when the request only justifies a standalone Project, Task, or Routine. It must not force every request into a Goal hierarchy.

---

## 3. Entry Points

### 3.1 Global AI Planning Entry

A primary action such as:

```txt
Plan with AI
```

opens AI planning without requiring existing Goal or Project context.

If an active unapproved conversation or draft already exists, the global entry must not silently replace it. The user first chooses:

```txt
Continue previous draft
Start new and discard previous draft
Cancel
```

The product may later support multiple saved drafts, but this discussion does not require it.

### 3.2 Contextual Entry from an Existing Goal or Project

From an existing Goal or Project, the user may choose:

```txt
Plan next steps with AI
```

The selected Goal or Project becomes explicit conversation context.

The AI must not silently move proposed items outside that context. If it believes another ownership model is more coherent, it must expose the change and request confirmation during draft review.

### 3.3 Manual Creation Entry

Manual creation remains permanently available outside the AI flow for:

- Goal
- Project
- Task
- Routine

Manual creation is a parallel path, not merely a failure fallback. The user must never be forced to use AI to create or edit canonical entities.

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

The UI must make two facts clear before submission:

```txt
The AI may ask a small number of questions.
Nothing is created until the user reviews and approves the draft.
```

---

## 5. Conversation Style

The interaction is hybrid:

```txt
conversational input
+ structured quick replies when useful
+ free-text escape at every clarification step
```

The AI may use buttons, chips, date pickers, or short option lists for bounded questions such as:

- target date
- preferred frequency
- available days
- ownership under an existing Goal or Project

The user is never limited to predefined options when they do not fit.

The system should prefer one question at a time unless two tightly coupled values are easier to answer together.

The flow must not become a long form disguised as a chat, one of software’s less charming methods of pretending friction is innovation.

---

## 6. Conversation States

### 6.1 Main Flow

```txt
EMPTY
→ INTERPRETING
→ CLARIFYING | DRAFT_READY

CLARIFYING
→ CLARIFYING | DRAFT_READY | INPUT_BLOCKED | CANCELLED

DRAFT_READY
→ REVIEWING | GENERATION_FAILED | CANCELLED

REVIEWING
→ DRAFT_READY | APPROVED | CANCELLED
```

The direct `INTERPRETING → DRAFT_READY` transition is required when the initial request is already specific enough.

### 6.2 Recovery Flow

```txt
INTERPRETING | CLARIFYING
→ INPUT_BLOCKED
→ CLARIFYING | DRAFT_READY | MANUAL_FALLBACK | CANCELLED

INTERPRETING | CLARIFYING | DRAFT_READY
→ GENERATION_FAILED
→ RETRYING | MANUAL_FALLBACK | CANCELLED

RETRYING
→ INTERPRETING | DRAFT_READY | GENERATION_FAILED

MANUAL_FALLBACK
→ manual creation path
```

### 6.3 State Meanings

#### EMPTY

No meaningful user intention has been submitted.

#### INTERPRETING

The system is identifying likely Goals, Projects, Tasks, Routines, ownership, and missing information.

#### CLARIFYING

The system asks only questions required to produce a useful and reviewable draft.

An out-of-scope or high-stakes request remains in `CLARIFYING` after the system states the boundary and offers a supported planning direction. The conversation waits for a revised or planning-relevant user input. The user may also cancel or continue manually.

No separate safety state is introduced here. Discussion 018 defines detailed safety behavior.

#### INPUT_BLOCKED

The system cannot proceed coherently because the input is too vague, contradictory, unsupported, or missing a required planning constraint.

#### DRAFT_READY

A draft has been generated, but no canonical product entity has been created.

#### REVIEWING

The user is editing, removing, accepting, or rejecting proposed items. Detailed draft editing and approval behavior belongs to Discussion 014.

#### APPROVED

The user has explicitly approved creation of the accepted draft content.

#### CANCELLED

The user exits without creating the proposed entities.

#### GENERATION_FAILED

The draft could not be produced because of a model, provider, network, parsing, or internal failure.

#### RETRYING

The system is making a user-requested retry using the preserved conversation input. Retry limits and provider behavior belong to Discussions 018 and 020.

#### MANUAL_FALLBACK

The AI conversation ends and transfers the understood user input or draft context into the relevant manual creation path without creating canonical entities automatically.

---

## 7. Clarification Policy

Clarification is selective, not mandatory for every request.

The AI should generate a draft immediately when the request is specific enough to produce a useful proposal without inventing important facts.

Example:

```txt
User: Create a routine to practice English for 30 minutes every weekday.
Expected: generate a draft directly.
```

The AI asks a clarification only when the answer materially changes:

- whether the intention is a Goal, Project, Task, or Routine
- ownership under an existing Goal or Project
- the actionable content of the draft
- timing or recurrence required to make the item meaningful
- user constraints that prevent an unrealistic or contradictory proposal

---

## 8. Mandatory, Optional, and Forbidden Questions

### 8.1 Mandatory Clarification

A question is mandatory only when the AI cannot generate a coherent draft without making a high-impact assumption.

Typical cases:

1. a recurring behavior is requested but no usable recurrence is provided
2. a date-sensitive request contains conflicting or impossible timing
3. an existing Goal or Project is referenced ambiguously
4. two materially different intentions cannot safely be combined or separated
5. the request depends on an explicit but undefined hard constraint

### 8.2 Optional Clarification

Optional questions may improve the draft but are not required to produce one.

Examples:

- preferred intensity
- available time per day
- preferred working days
- desired target date
- whether a standalone item should later connect to a Goal

The AI may ask an optional question only when its expected value justifies another interaction step.

### 8.3 Forbidden Clarification

The AI must not ask for:

- sensitive personal data not required for planning
- a complete life history
- psychological diagnosis
- motivation-scoring questionnaires before planning
- information already provided in the conversation or known context
- implementation details the user should not need to understand
- confirmation of obvious low-impact assumptions that can be shown in the draft

The AI must not repeatedly ask differently worded versions of the same unresolved question.

---

## 9. Clarification Limit

The default clarification budget is:

```txt
maximum 3 AI clarification turns before producing a draft
```

A clarification turn may contain one question or one tightly coupled question group. Most flows should need zero to two turns.

After the third turn, the AI must do one of the following:

```txt
1. generate a draft with clearly labeled assumptions
2. generate a smaller partial draft for the understood portion
3. explain the single blocking ambiguity and offer manual creation or restart
```

It must not continue an open-ended interview.

The limit resets only when:

- the user materially changes the original intention
- the user explicitly restarts
- the user approves splitting the request into separate planning flows

---

## 10. Draft Now

At any clarification step, the user may choose:

```txt
Draft now
```

The AI then generates the best coherent draft using:

- explicit user input
- selected Goal or Project context
- low-risk assumptions

Every material assumption must be visible in the draft.

The AI must not silently invent:

- deadlines
- recurrence schedules
- ownership relationships
- measurable Goal outcomes
- user availability

If one of these is essential and cannot be safely assumed, the AI may produce a partial draft rather than fabricate a complete one.

---

## 11. Direct Narrow Requests

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
→ standalone Project with Tasks, when the finite effort justifies a Project
```

Discussion 012 ownership rules remain unchanged:

```txt
Task or Routine may belong to one Goal, one Project, or neither.
Task or Routine may not belong to Goal and Project simultaneously.
```

---

## 12. Vague, Contradictory, or Incomplete Input

### 12.1 Vague but Usable

For input such as:

```txt
I want to get better at English.
```

The AI should ask one high-value question or generate a conservative draft with visible assumptions.

### 12.2 Too Vague to Plan

For input such as:

```txt
Fix my life.
```

The AI should narrow the entry point without pretending it understands the entire problem.

A response may offer broad areas while preserving free text:

```txt
Which area feels most important to improve first?
- work or study
- health
- personal organization
- relationships
- something else
```

### 12.3 Contradictory Input

For input such as:

```txt
Create a daily routine, but I only want to do it once.
```

The AI must identify the contradiction and ask the smallest resolving question. It must not silently choose one side.

### 12.4 Incomplete but Non-Blocking

If missing information can be represented as an editable assumption, the AI should produce the draft rather than prolong clarification.

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

For requests involving medical treatment, mental-health crisis, legal strategy, financial investment, physical danger, or other high-stakes decisions:

```txt
The AI may organize user-provided actions or questions.
It must not replace qualified professional judgment.
It must not invent expert instructions.
```

Detailed safety language and refusal boundaries belong to Discussion 018.

Capability boundary:

> The system may structure ordinary intentions into planning entities, but it must not act as an unqualified domain expert.

---

## 14. Out-of-Scope Requests

When a request falls outside planning capability, the system should:

1. state the boundary briefly
2. preserve any planning-relevant portion
3. offer the nearest supported planning action
4. remain in `CLARIFYING` and wait for revised planning-relevant input, unless the user cancels or continues manually

Example:

```txt
User: Diagnose my symptoms and create a treatment plan.

Supported response direction:
I cannot diagnose or prescribe treatment. I can help you prepare questions for a clinician, track appointments, or organize actions already recommended by a qualified professional.
```

The system must not create Tasks or Routines that imply unsupported professional advice.

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

Clears the current planning conversation and returns to `EMPTY`.

### Edit Original Intention

Returns to the initial input. Existing conversation content is discarded unless the user explicitly chooses to preserve it.

### Continue Manually

Transitions to `MANUAL_FALLBACK` and opens the relevant manual creation path.

### Context Change

If the user materially changes the intention during clarification, the AI must acknowledge the change and either:

```txt
continue as one revised flow
or propose splitting it into separate drafts
```

It must not quietly blend unrelated intentions into one hierarchy.

---

## 16. Session Recovery

If the user leaves before approval, the product may preserve an unapproved conversation or draft for short-term recovery.

Rules:

- no Goal, Project, Task, or Routine is created before explicit approval
- recovery must clearly label the content as an unapproved draft
- the user may continue, discard and restart, continue manually, or cancel
- opening global AI planning while such a draft exists must show the collision choice defined in section 3.1
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

Partial user input should not be lost merely because generation failed.

The product must not claim entities were created unless creation actually succeeded after explicit approval.

Detailed error taxonomy, retry limits, idempotency, and provider fallback belong to Discussions 018–020.

---

## 18. Accepted End-to-End Flow

```txt
User opens AI planning
→ if an active unapproved draft exists, choose continue / discard and restart / cancel
→ enter a Goal, Project, Task, Routine, or rough intention
→ INTERPRETING

If already specific enough:
→ DRAFT_READY

If a high-impact ambiguity exists:
→ CLARIFYING
→ ask up to 3 clarification turns
→ user answers, chooses Draft now, restarts, cancels, or continues manually

If input is outside planning capability:
→ state boundary
→ remain in CLARIFYING for revised planning-relevant input

Then:
→ DRAFT_READY
→ REVIEWING
→ user edits, removes, approves, or cancels

Only after explicit approval:
→ APPROVED
→ canonical entities may be created
```

Core rule:

> Conversation produces a proposal. Approval creates product entities.

---

## 19. Accepted Decisions

### Entry Model

- The flow accepts a Goal, Project idea, Task, Routine, or free-form intention.
- A Goal is not required.
- Narrow requests may produce standalone entities.
- Entry is available globally and contextually from a Goal or Project.
- Manual creation remains permanently available as a parallel path.
- A new global entry cannot silently discard an active unapproved draft.

### Clarification Policy

- Clarification is selective.
- The AI asks only questions that materially affect the draft.
- The maximum default clarification budget is three turns.
- The user may request `Draft now` at any clarification step.
- Missing non-blocking details become visible draft assumptions.
- High-impact facts must not be silently invented.

### Interaction and State Model

- The flow is conversational with structured quick replies where useful.
- Free text remains available at every clarification step.
- `INTERPRETING` may transition directly to `DRAFT_READY`.
- Out-of-scope boundary responses remain in `CLARIFYING` pending revised input.
- `RETRYING` and `MANUAL_FALLBACK` are explicit recovery states.

### Domain Boundary

- The pilot supports ordinary personal planning across broad life domains.
- There is no brittle fixed topic allowlist in this discussion.
- The system structures plans but does not replace professional judgment in high-stakes domains.

### Exit and Failure Behavior

- The user may cancel, restart, edit the intention, draft immediately, or continue manually.
- No canonical entities are created before explicit approval.
- Failure preserves user input and offers retry, edit, manual fallback, or cancel.

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
Active-draft collision choice
Contextual Goal/Project entry
Manual creation path
Direct draft bypass
Clarification loop
Out-of-scope boundary return to clarification
Draft now path
Draft review boundary
Cancel/restart/manual fallback
Generation failure and retry recovery
```

### AI Guardrails

Add:

- no forced Goal hierarchy
- no silent high-impact assumptions
- no repeated questioning for known information
- no expert substitution in high-stakes domains
- no claim of creation before approval succeeds
- no silent replacement of an active unapproved draft

### Open Questions to Remove or Narrow

- whether the user must start with a Goal
- whether direct Task-only or Routine-only requests are supported
- whether the experience is chat or form
- whether manual creation exists
- whether clarification may be skipped
- what happens when global entry meets an active draft
- how out-of-scope requests map to conversation state

No Mind Map change is applied yet.

---

## 21. Documents Affected — Record Only, Do Not Update Yet

After Discussion 021 consolidation, accepted decisions from this discussion should update or create:

- AI planning conversation flow specification
- product UX flow documentation
- AI responsibility and guardrail specification
- draft review specification in coordination with Discussion 014
- failure, recovery, and safety specification in coordination with Discussion 018
- runtime/API specification in coordination with Discussion 020
- analytics event specification in coordination with Discussion 019
- implementation plan in Discussion 022

Potential ADR impact:

- an ADR may be required for the AI planning entry model and approval boundary if no existing ADR already locks them

No Mind Map or formal document should be updated from this discussion alone before consolidation after Discussion 021.

---

## 22. Review Resolution

Claude confirmed that the following decisions are coherent without further change:

- maximum three-turn clarification budget
- manual creation as a permanent parallel path
- planning capability boundary instead of a hard topic allowlist
- prohibition on silently inventing high-impact values
- broad and narrow AI entry intentions
- explicit approval before canonical entity creation

Accepted findings and resolutions:

```txt
F1 — Important
Out-of-scope behavior lacked a formal state mapping.
Resolved: boundary response remains in CLARIFYING and awaits revised planning-relevant input.
Discussion 018 must preserve this transition while defining detailed safety behavior.

F2 — Important
The state diagram omitted direct draft generation.
Resolved: INTERPRETING may transition directly to DRAFT_READY.

F3 — Important
Global entry could collide ambiguously with an active unapproved draft.
Resolved: user chooses continue previous draft, discard and start new, or cancel.

F4 — Minor
RETRYING and MANUAL_FALLBACK appeared without definitions.
Resolved: both states are explicitly defined.
```

No blocking issues remain. Discussion 013 is accepted and closed for the current review sequence.