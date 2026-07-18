# Discussion 014 — AI Draft Structure and Approval Flow

## Status

Open for GPT × Claude review.

This document contains GPT's proposed decisions. It must not be marked accepted until Claude reviews this exact version, Mahdi resolves any product choices, accepted corrections are integrated, and the final resolution, Mind Map Impact, and Affected Formal Documents are complete.

## Accepted Dependencies

This discussion depends on:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]

Accepted constraints carried forward:

- Canonical concepts are `Goal`, `Project`, `Task`, `Routine`, and `RoutineOccurrence`.
- `Plan` is not a persisted core entity.
- AI output is ephemeral until explicit user approval.
- Conversation produces a proposal; approval creates canonical entities.
- A Task or Routine may belong to one Goal, one Project, or neither, but never both.
- AI must not silently invent high-impact values such as deadlines, recurrence, ownership, measurable Goal outcomes, or user availability.
- Crisis input routes to the fixed safety fallback from Discussion 013 and does not produce a planning draft.

---

## 1. Scope

This discussion defines the product-level structure of an AI-generated planning draft and the user approval flow that turns accepted draft items into canonical entities.

It answers:

```txt
What does a draft contain?
How are proposed entities and relationships represented to the user?
What may the user edit, remove, or approve?
Is approval all-or-nothing or selective?
When does creation occur?
What happens if creation partially fails?
What remains ephemeral after approval or cancellation?
```

It does not define:

- exact provider prompt or model-specific response format
- database tables, transactions, idempotency, or event schema — Discussion 019
- runtime APIs, provider architecture, parsing retries, or cost controls — Discussion 020
- Today scheduling and execution behavior — Discussion 015
- Reconcile proposals — Discussions 016 and 017
- detailed privacy, retention, and provider-failure policy — Discussion 018

---

## 2. Included Now / Excluded / Deferred

### Included Now

- conceptual draft structure
- supported proposed entity types
- temporary references and relationships inside a draft
- assumptions and warnings visible to the user
- editing and selective approval behavior
- validation before creation
- creation boundary and user-visible success/failure result
- handling stale context before approval

### Explicitly Excluded

- persisted `Plan` entity
- AI auto-approval
- silent creation before review
- Task dependency graphs
- generic canonical Task ordering
- approval based only on a conversational phrase when the reviewed draft has changed since it was shown
- automatic application of draft changes to existing canonical entities

### Deferred

- exact JSON Schema and field names used between model and backend
- transaction and rollback implementation
- analytics events
- retention duration for abandoned drafts
- collaborative or multi-user approval
- version history UI for draft revisions

---

## 3. Draft as an Ephemeral Proposal

A planning draft is a temporary proposal, not a canonical product entity.

```txt
User intention
→ AI conversation
→ Draft
→ user review and edits
→ explicit approval
→ canonical Goal / Project / Task / Routine creation
```

The draft may be temporarily persisted for recovery, as allowed by Discussion 013, but that persistence does not make it an accepted plan or canonical domain concept.

A draft has its own temporary identity only for review, editing, retry, and recovery. That identity must not be exposed as a permanent product object after completion.

---

## 4. Proposed Draft Contents

A draft contains four user-facing parts:

```txt
1. Summary
2. Proposed entities
3. Proposed relationships
4. Assumptions, warnings, and unresolved items
```

### 4.1 Summary

The summary briefly states how the AI interpreted the intention.

Example:

> I interpreted this as one Goal, one finite Project, two Tasks, and one supporting Routine.

The summary is explanatory only. It does not create a separate entity and must not make unsupported claims about the user's motivation, personality, or likelihood of success.

### 4.2 Proposed Entities

The draft may propose:

- zero or more Goals
- zero or more Projects
- zero or more Tasks
- zero or more Routines

`RoutineOccurrence` is not proposed directly in this discussion. It belongs to scheduling and execution behavior in Discussion 015.

Each proposed entity contains only the conceptual information needed for the user to understand and approve it.

Common proposed fields:

```txt
Temporary draft item ID
Entity type
Title
Optional description or purpose
Proposed parent reference, if any
Type-specific planning information
Visible assumptions or warnings
```

The temporary draft item ID exists only to connect draft items and user edits. It is not a canonical entity ID.

### 4.3 Type-Specific Information

#### Goal

May include:

- title
- optional description of the desired outcome
- optional user-provided target date
- any explicit outcome wording that still requires user confirmation in the real world

The AI must not present a generated metric as proof of Goal achievement.

#### Project

May include:

- title
- optional description of the finite effort
- optional user-provided target date
- optional parent Goal reference

#### Task

May include:

- title
- optional description
- optional date or planning placement when explicitly provided or safely resolved
- optional parent Goal or Project reference

A temporary suggested sequence may be shown in the draft presentation, but it does not become a generic canonical Task `order` field.

#### Routine

May include:

- title
- optional description
- proposed recurrence
- optional duration or expected effort when explicitly provided
- optional parent Goal or Project reference

A Routine that lacks enough recurrence information to be meaningful must remain unresolved or be removed before approval. The system must not silently invent a recurrence schedule.

### 4.4 Proposed Relationships

Relationships inside the draft must conform to Discussion 012 before approval.

Allowed:

```txt
Project → one Goal or standalone
Task → one Goal, one Project, or standalone
Routine → one Goal, one Project, or standalone
```

Forbidden:

```txt
Task → Goal and Project simultaneously
Routine → Goal and Project simultaneously
Task or Routine → multiple parents
Task dependency relationships
```

The draft review UI should explain relationships in ordinary language rather than exposing storage terminology.

---

## 5. Assumptions, Warnings, and Unresolved Items

### 5.1 Visible Assumptions

A low-risk assumption may appear in the draft when it allows useful progress without forcing another clarification turn.

Each material assumption must be attached to the relevant item or relationship.

Example:

```txt
Assumption: This Task is standalone because no Goal or Project was specified.
```

The user may accept, edit, or remove the assumption by changing the affected draft structure.

### 5.2 Warnings

Warnings explain a structural or planning concern without blocking approval when the structure remains valid.

Examples:

- several Tasks may be unrealistic for the same day
- a Project has no proposed actionable work yet
- a Goal contains activity but no explicit user-defined outcome wording

Warnings must be non-judgmental and must not shame the user.

### 5.3 Unresolved Items

An unresolved item represents missing high-impact information that cannot be safely guessed.

Examples:

- Routine recurrence is missing
- parent context is ambiguous
- contradictory dates remain unresolved

A draft with unresolved blocking items may be reviewed and edited, but it cannot be fully approved until each blocking item is resolved or the affected proposed item is removed.

---

## 6. Draft Review Model

The user reviews the proposed hierarchy before anything is created.

The review should support:

```txt
Edit item
Change item type when structurally valid
Change parent relationship
Remove item
Restore a recently removed item during the current review session
Add a manual item to the draft
Regenerate a selected part
Regenerate the whole draft
Approve selected valid items
Approve all valid items
Cancel
```

The review UI must clearly distinguish:

- AI-proposed content
- user-edited content
- unresolved assumptions or warnings
- items selected for approval

The UI does not need to preserve authorship metadata after canonical creation unless a later analytics decision defines a real consumer for it.

---

## 7. Editing Rules

### 7.1 User Edits Take Priority

Once the user edits a draft item, later AI regeneration must not silently overwrite that edit.

A regeneration affecting user-edited content must either:

```txt
preserve the edit
or show the proposed replacement and request confirmation
```

### 7.2 Type Changes

The user may change an item's type when the resulting structure is coherent.

Examples:

```txt
Task → Project
Routine → Task
Project → Task
```

Changing type may invalidate recurrence, child relationships, or ownership. The UI must surface the consequences before applying the type change.

### 7.3 Parent Changes

The user may attach or detach a proposed Project, Task, or Routine from an allowed parent.

The exclusive-parent invariant must be enforced immediately in the draft, not postponed until canonical creation.

### 7.4 Removing a Parent

If the user removes a proposed Goal or Project that has selected children, the product must not silently delete or reparent those children.

The user must choose among the structurally valid options:

```txt
Remove parent and selected children
Keep children as standalone items where allowed
Move children to another valid proposed parent
Cancel removal
```

Projects may become standalone when their Goal is removed.
Tasks and Routines may become standalone when their Goal or Project is removed.

---

## 8. Selective Approval

Approval is selective rather than strictly all-or-nothing.

Reasons:

- a useful draft may contain one weak proposal among several valid ones
- forcing all-or-nothing approval creates unnecessary regeneration and friction
- the accepted core model allows standalone entities, so deselecting a parent does not always require discarding every child

The user may approve:

```txt
all valid selected items
or a coherent valid subset
```

A selected subset must still satisfy all structural invariants.

Examples:

- approving a Project without its proposed Goal creates a standalone Project, after explicit user-visible confirmation
- approving a Task without its proposed parent creates a standalone Task, after explicit confirmation
- approving child Tasks while rejecting their Project requires explicit reparenting or standalone confirmation

The system must not silently reinterpret ownership merely to make a selected subset valid.

---

## 9. Approval Boundary

Approval must refer to a specific visible draft revision.

A valid approval requires:

```txt
1. the user has been shown the current draft revision
2. selected items are clearly identified
3. blocking validation errors are absent
4. the user performs an explicit approval action
```

Examples of explicit approval actions:

- pressing `Create selected items`
- pressing `Create all`
- an equivalent accessible confirmation control

A generic conversational phrase such as “looks good” may move the UI toward confirmation, but it must not create entities if the visible draft changed after the phrase or if the selected scope is ambiguous.

Any draft-changing action invalidates a prior unexecuted approval intent.

---

## 10. Pre-Creation Validation

Before creation, the system validates the selected draft deterministically.

Validation includes:

- supported entity types only
- required user-facing title or equivalent meaningful content
- allowed ownership relationships
- exclusive-parent rule
- valid recurrence for each selected Routine
- no selected child references an unselected parent unless standalone or reparenting was explicitly confirmed
- no blocking unresolved item remains
- selected context still exists and remains eligible when the draft originated from an existing Goal or Project

AI must not be the final authority for these invariants.

---

## 11. Stale Context and Concurrent Change

A draft may become stale before approval because an existing Goal or Project was changed, completed, stopped, or deleted elsewhere.

Before creation, the system must re-check contextual references.

If a referenced parent is no longer valid, the system pauses creation and offers only coherent choices:

```txt
choose another valid parent
create affected items as standalone where allowed
remove affected items
cancel
```

The system must not silently attach items to stale or different context.

Detailed concurrency and transaction behavior belongs to Discussions 019 and 020.

---

## 12. Creation Result

### 12.1 Success

After successful creation, the product shows what was created.

Example:

```txt
Created:
- 1 Goal
- 1 Project
- 3 Tasks
- 1 Routine
```

The user may navigate to the created context or continue to Today/planning behavior defined later.

### 12.2 Failure

The product must not claim success unless canonical creation actually succeeded.

At the product level, the selected approval is one creation operation. The intended outcome is:

```txt
all selected canonical entities and relationships are created
or no selected entity is reported as created
```

Atomic transaction and idempotency mechanics belong to Discussion 019.

If creation fails:

- preserve the approved draft revision
- show a recoverable failure state
- allow retry
- do not require the user to repeat review unless the draft or context changed
- do not silently create a different subset

### 12.3 Duplicate Retry Protection

Retrying an uncertain creation result must not produce duplicate canonical entities.

The implementation mechanism is deferred, but this is a required product invariant for Discussions 019 and 020.

---

## 13. Existing Entity Changes

This discussion primarily covers creation of new entities.

If the AI proposes changes to existing canonical entities during the planning flow:

- those changes must be displayed separately from new-item creation
- each consequential change requires explicit approval
- unchanged existing entities must not be recreated
- the system must not mix creation and destructive edits into an ambiguous single confirmation

The detailed update-action model is deferred to Discussion 017 for Reconcile and to later product specifications where ordinary editing is defined.

For the first planning flow, proposing new entities is included; bulk mutation of existing canonical entities is excluded.

---

## 14. Cancellation and Draft Lifecycle

Before creation, the user may cancel without creating canonical entities.

Cancellation may:

- discard the draft immediately
- or preserve it temporarily for recovery, according to Discussions 013 and 018

A cancelled or expired draft must never reappear as accepted canonical work.

After successful creation, the ephemeral draft may retain only the minimum information required for confirmation, troubleshooting, or defined analytics. Detailed retention belongs to Discussions 018 and 019.

---

## 15. Scenario Checks

### Scenario A — Simple standalone Routine

```txt
User: Practice English for 30 minutes every weekday.
Draft: one standalone Routine with weekday recurrence.
Approval: Create Routine.
Expected: one Routine is created; no Goal or Project is invented.
```

### Scenario B — Goal with direct work and Project

```txt
Goal: Find a frontend job
Direct Routine: Practice interview answers three times per week
Project: Build portfolio
Task under Project: Publish case study
```

Expected:

- ownership is visible
- Routine may remain directly under Goal
- Task belongs only to Project
- Project completion does not prove Goal achievement

### Scenario C — Reject parent, keep child

```txt
Draft proposes Goal + standalone-capable Task.
User rejects Goal and keeps Task.
```

Expected:

- system asks for explicit standalone confirmation
- Task is not silently attached elsewhere

### Scenario D — Remove Project with children

Expected:

- user chooses remove children, preserve them standalone, or reparent them
- no silent deletion or reparenting

### Scenario E — Missing Routine recurrence

Expected:

- Routine is marked unresolved
- other valid items may still be selectively approved
- unresolved Routine cannot be created until fixed or removed

### Scenario F — Context completed before approval

```txt
Draft originated under Project A.
Project A is completed in another session before approval.
```

Expected:

- stale context is detected
- creation pauses
- user chooses standalone, another parent, removal, or cancel

### Scenario G — Retry after uncertain failure

Expected:

- reviewed draft remains available
- retry cannot create duplicates
- UI does not claim partial success without a verified result

### Scenario H — Crisis language appears during review

Expected:

- normal review flow stops
- Discussion 013 `SAFETY_FALLBACK` takes precedence
- no draft item is created from crisis content

---

## 16. Proposed Decisions

### Draft Meaning

- A draft is an ephemeral proposal, not a persisted `Plan` domain entity.
- It may have a temporary recovery identity without becoming canonical.
- It contains summary, proposed entities, relationships, assumptions, warnings, and unresolved items.

### Editing

- The user may edit, remove, add, reparent, and regenerate draft content.
- User edits are not silently overwritten by later AI output.
- Parent removal requires explicit handling of children.

### Approval

- Approval is selective and revision-specific.
- Only a coherent valid subset may be approved.
- Ownership changes caused by selective approval require explicit confirmation.
- A generic conversational acknowledgment is not sufficient when scope or revision is ambiguous.

### Validation and Creation

- Structural invariants are validated deterministically before creation.
- Context references are rechecked before creation.
- Creation is presented as one atomic product operation.
- Failure preserves the approved draft and supports duplicate-safe retry.

### Boundaries

- RoutineOccurrence creation is outside this discussion.
- Exact model JSON, persistence, transactions, APIs, and analytics are deferred.
- Bulk AI mutation of existing entities is excluded from the initial planning creation flow.

---

## 17. Mind Map Impact — Record Only, Do Not Apply Yet

After acceptance, later consolidation should add or revise:

### AI Responsibilities

- produce reviewable draft entities that map only to the accepted core model
- expose assumptions, warnings, and unresolved high-impact fields
- preserve user edits during regeneration
- never create canonical entities directly

### User Flow

```txt
DRAFT_READY
→ REVIEWING
→ edit / remove / add / reparent / regenerate
→ select coherent subset
→ deterministic validation
→ explicit revision-specific approval
→ create all selected items or report failure
```

Add stale-context resolution and duplicate-safe retry to the approval path.

### AI Guardrails

- no hidden parent changes
- no silent overwrite of user edits
- no approval of unresolved blocking items
- no success claim before verified canonical creation
- safety fallback overrides draft review

### Open Questions to Remove or Narrow

- whether approval is all-or-nothing
- whether users may edit individual proposals
- whether rejecting a parent silently removes children
- whether a conversational “looks good” automatically creates entities
- whether stale context is checked before creation

No Mind Map change is applied yet.

---

## 18. Affected Formal Documents — Record Only, Do Not Update Yet

After Discussion 021 consolidation, accepted decisions should update or create:

- AI draft/output contract specification
- draft review UX specification
- canonical entity creation and approval specification
- validation and ownership invariant specification
- safety precedence specification in coordination with Discussion 018
- transaction, idempotency, and event specification in Discussion 019
- runtime/API and retry specification in Discussion 020
- analytics specification in Discussion 019/021 where justified
- implementation plan in Discussion 022

Potential ADR impact:

- an ADR may be needed for selective approval and revision-specific approval
- an ADR may be needed for atomic creation of an approved selected subset

No formal document is updated from this discussion alone.

---

## 19. Structured Review Request for Claude

Review this exact proposal using `02-Decisions/claude-review-standards.md`.

### Review Objective

Find only:

1. contradictions with Discussions 012 or 013
2. draft structures that cannot map coherently to the accepted core model
3. missing review, editing, approval, cancellation, or recovery states
4. selective-approval cases that silently change ownership or lose user work
5. ambiguous approval boundaries
6. stale-context or retry cases that can create duplicates or misleading success
7. unsupported safety behavior or responsibility leaked into later discussions
8. the smallest coherent corrections required

### Do Not

- redesign the product from zero
- replace selective approval merely because all-or-nothing is cheaper
- invent database tables, API routes, provider choices, token budgets, or prompt text
- add Task dependency graphs or a generic canonical Task order field
- turn the ephemeral draft into a persisted `Plan` entity
- expand into Today, Reconcile, or implementation sequencing

### Required Response Format

For each finding:

```txt
Finding ID:
Severity: Blocking | Important | Minor
Affected decision:
Concrete failure scenario:
Why it conflicts or fails:
Smallest coherent correction:
Affected earlier decision or later discussion:
```

Then finish with:

```txt
A. Blocking issues
B. Important corrections
C. Minor corrections
D. Coherent decisions confirmed as written
E. Minimal revised decision list
F. Mind Map impact changes
G. Affected formal document changes
```

Do not provide a replacement product design. Preserve coherent decisions and change only what is necessary.
