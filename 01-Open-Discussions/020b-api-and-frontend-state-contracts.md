# Discussion 020B — API and Frontend State Contracts

## Status

Open for GPT × Claude review.

This document is Part B of Discussion 020.

Companion discussions:

- [[01-Open-Discussions/020-api-and-ai-runtime-architecture]]
- [[01-Open-Discussions/020a-final-ai-runtime-boundaries-and-orchestration-resolution]]
- [[01-Open-Discussions/020c-structured-output-reliability-and-cost-controls]]

Accepted dependencies:

- [[01-Open-Discussions/019a-final-canonical-data-model-resolution]]
- [[01-Open-Discussions/019b-final-transactions-concurrency-and-idempotency-resolution]]
- [[01-Open-Discussions/019c-final-events-ai-observability-and-retention-resolution]]
- [[01-Open-Discussions/019c-amendment-ai-context-scope-observability]]
- [[01-Open-Discussions/020a-final-ai-runtime-boundaries-and-orchestration-resolution]]

---

## 1. Scope

This discussion defines the transport-facing contracts used by web and future clients to:

- start and observe Planning AI attempts
- retrieve validated PlanningDraft revisions
- start and observe Reconcile sessions
- receive deterministic facts and rule matches
- receive optional AI explanations
- edit proposal items without mutating canonical state
- request and submit explicit confirmations
- execute deterministic commands
- recover from stale versions and expired confirmations
- cancel in-flight AI attempts
- preserve user-authored input after failure
- represent degraded mode without inventing partial success

The API exposes accepted product behavior. It does not redefine domain semantics.

---

## 2. Explicitly Out of Scope

This discussion does not define:

- provider SDK calls
- prompt construction
- structured-output repair internals
- timeout duration values
- retry counts
- circuit-breaker thresholds
- final URL naming conventions
- UI styling or component hierarchy
- authentication protocol choice
- implementation sequencing
- offline-first synchronization
- webhook or push-notification delivery

Those belong to Discussions 020C, 021, and 022.

---

## 3. Governing API Principles

### 3.1 Server Authority

The client may submit:

- user-authored text
- selected proposal item IDs
- explicit edits
- expected entity versions
- confirmation identity
- idempotency key

The client may not authoritatively submit:

- computed consequences
- current lifecycle eligibility
- ownership relationships
- protection status
- final mutation set inferred from stale data
- rule matches
- safety classification

The server reloads and revalidates all canonical facts before mutation.

### 3.2 Proposal Is Not Mutation

```txt
PlanningDraft / ReconcileRecommendation
→ reviewable proposal
→ no canonical mutation

ActionConfirmation
→ explicit user approval of one exact preview

CommandResult
→ authoritative mutation outcome
```

Acceptance does not prove execution success.

### 3.3 No Partial AI Success

A client never receives an approvable partial PlanningDraft.

```txt
provider partial output
schema-invalid output
policy-invalid output
cancelled late output
→ no draft revision exposed
```

### 3.4 Stable Resource Identity

Every long-lived API resource has a server-generated immutable ID.

Client-generated temporary IDs may be used only inside user-authored draft edits before server acceptance and may never become canonical entity IDs.

---

## 4. Resource Families

The proposed MVP API exposes these logical resources:

```txt
PlanningAttempt
PlanningDraft
PlanningDraftRevision
ReconcileSession
ReconcileRecommendation
ActionConfirmation
CommandSubmission
CommandResult
```

`CommandSubmission` may be transport-only rather than persisted as a separate canonical record. The durable idempotency and audit records remain defined by Discussion 019.

---

## 5. Common Envelope

Every API response uses a common top-level envelope.

```txt
ApiResponse<T>
- requestId
- correlationId
- serverTime
- data?
- error?
- warnings[]
```

Rules:

- exactly one of `data` or `error` is present
- `requestId` identifies the transport request
- `correlationId` links related attempt, confirmation, command, and event records
- warnings never replace errors
- warnings may not imply that a failed mutation partially succeeded

---

## 6. Planning Attempt Contract

### 6.1 Start Planning Attempt

Logical operation:

```txt
POST /planning-attempts
```

Request:

```txt
StartPlanningAttemptRequest
- userInput
- timezone
- localDate
- clientAttemptId?
- sourceSurface
- continuationOfAttemptId?
```

The request must not include raw canonical entity snapshots supplied as authority.

The server builds context through the accepted Planning context builder.

Response:

```txt
StartPlanningAttemptResponse
- attemptId
- status
- createdAt
- pollingUrl
- cancellationAllowed
```

Accepted initial statuses:

```txt
QUEUED
RUNNING
COMPLETED
FAILED
CANCELLED
```

A synchronous fast completion may return `COMPLETED`, but clients must support `QUEUED` or `RUNNING`.

### 6.2 Planning Transport Choice

MVP proposal:

```txt
start request: synchronous resource creation
result delivery: polling
text-token streaming: disabled
server-sent progress: deferred
```

Reason:

- Planning output is not valid until complete validation succeeds
- token streaming would expose non-authoritative intermediate content
- polling is simpler for reconnect, mobile clients, and late completion

### 6.3 Retrieve Attempt

```txt
GET /planning-attempts/{attemptId}
```

Response:

```txt
PlanningAttemptView
- attemptId
- status
- createdAt
- startedAt?
- completedAt?
- cancelledAt?
- failure?
- latestDraftId?
- latestRevisionNumber?
- retryAllowed
- manualFallbackAvailable
```

Failure details must be user-safe and must not expose provider internals.

### 6.4 Cancel Attempt

```txt
POST /planning-attempts/{attemptId}/cancel
```

Request:

```txt
CancelAttemptRequest
- expectedAttemptVersion
- clientReason: USER_CANCELLED | NAVIGATION | REPLACED_BY_NEW_ATTEMPT
```

Result:

```txt
CANCELLED
ALREADY_COMPLETED
ALREADY_CANCELLED
CONFLICT
```

MVP invariant from 020A:

```txt
late provider result after accepted cancellation
→ operationally recorded
→ never exposed as a draft
→ never attached as a new revision
```

Cancellation is idempotent.

---

## 7. PlanningDraft Contract

### 7.1 Draft Creation Boundary

A PlanningDraft becomes retrievable only after:

- provider completion
- parsing
- schema validation
- deterministic repair within allowlist, if any
- policy validation
- context-bound validation
- complete-output validation

The client never sees a half-valid draft.

### 7.2 Draft View

```txt
PlanningDraftView
- draftId
- attemptId
- revisionNumber
- status
- createdAt
- expiresAt
- items[]
- warnings[]
- consequencePreview
- confirmationEligibility
- sourceMetadata
```

Draft status:

```txt
REVIEWABLE
SUPERSEDED
EXPIRED
CONFIRMED
CANCELLED
```

`CONFIRMED` means a confirmation was submitted for that exact revision. It does not itself prove command success.

### 7.3 Draft Item

```txt
PlanningDraftItemView
- itemId
- itemType: GOAL | PROJECT | TASK | ROUTINE
- proposedFields
- proposedParentReference?
- provenance
- validationWarnings[]
- editableFields[]
- selectedByDefault
```

The proposal may include temporary item references between new draft items.

Canonical entity IDs are absent until command commit succeeds.

### 7.4 Draft Revisions

Editing a draft creates a new immutable revision.

```txt
PATCH /planning-drafts/{draftId}
```

Request:

```txt
EditPlanningDraftRequest
- baseRevisionNumber
- edits[]
```

Every edit identifies:

```txt
- itemId
- field
- newValue
```

Response:

```txt
new revisionNumber
new consequencePreview
new confirmationEligibility
```

Rules:

- old revisions remain historical and non-confirmable after supersession
- edits never mutate canonical entities
- client may not submit a replacement consequence preview
- server regenerates preview from edited draft data

### 7.5 Draft Staleness

A draft may become stale when:

- referenced canonical entity version changes
- ownership changes
- protection changes
- relevant deadline/checkpoint changes
- draft expires
- a newer revision supersedes it

The API exposes:

```txt
confirmationEligibility:
  ELIGIBLE
  STALE_REFRESH_REQUIRED
  EXPIRED
  BLOCKED_BY_POLICY
```

---

## 8. Reconcile Session Contract

### 8.1 Start Reconcile

```txt
POST /reconcile-sessions
```

Request:

```txt
StartReconcileRequest
- localDate
- timezone
- requestedScope?
```

`requestedScope` is a bounded product-defined selector, not arbitrary query language.

Response:

```txt
ReconcileSessionView
- sessionId
- status
- factsStatus
- explanationStatus
- createdAt
- pollingUrl
```

### 8.2 Reconcile Processing Model

```txt
deterministic facts and rule matches
→ mandatory

AI explanation
→ optional enrichment
```

Session statuses:

```txt
RUNNING
FACTS_READY
COMPLETED
FAILED
CANCELLED
```

Explanation statuses:

```txt
NOT_REQUESTED
PENDING
AVAILABLE
UNAVAILABLE
CANCELLED
```

A session may be usable in degraded mode when:

```txt
factsStatus = AVAILABLE
explanationStatus = UNAVAILABLE
```

### 8.3 Deterministic Failure

If deterministic Reconcile fails:

```txt
status = FAILED
factsStatus = UNAVAILABLE
explanationStatus = NOT_REQUESTED
```

The client must display Reconcile unavailable.

The server must not ask AI to summarize raw data as a fallback.

### 8.4 Reconcile View

```txt
ReconcileSessionDetail
- sessionId
- status
- observedAt
- facts[]
- ruleMatches[]
- groups[]
- recommendations[]
- explanation?
- degradedMode
- retryAllowed
- manualFallbackAvailable
```

Every recommendation contains:

```txt
- recommendationId
- groupId
- actionType
- affectedEntityIds[]
- reasonCodes[]
- evidenceQuality
- preview
- outcome
- resultingCommandId?
- resultingCommandStatus?
```

### 8.5 Accepted but Not Applied

The API explicitly distinguishes:

```txt
recommendationOutcome = ACCEPTED
resultingCommandStatus = CONFLICTED
```

or:

```txt
recommendationOutcome = ACCEPTED_EDITED
resultingCommandStatus = SUCCEEDED
```

The client must never render `ACCEPTED` alone as “Applied.”

---

## 9. Confirmation Contract

### 9.1 Create Confirmation Preview

The client requests a server-authoritative confirmation resource.

```txt
POST /action-confirmations
```

Request:

```txt
CreateConfirmationRequest
- sourceType: PLANNING_DRAFT | RECONCILE_RECOMMENDATION | MANUAL_ACTION
- sourceId
- sourceRevision?
- selectedItemIds[]
- explicitEdits[]?
```

The server loads current canonical state and returns:

```txt
ActionConfirmationView
- confirmationId
- confirmationVersion
- status
- expiresAt
- actionSummary
- selectedEntityOrItemIds[]
- expectedEntityVersions[]
- consequences[]
- warnings[]
- reversible
- confirmationRequired
```

Confirmation statuses:

```txt
PENDING
CONFIRMED
EXPIRED
INVALIDATED
CONSUMED
CANCELLED
```

### 9.2 Client Trust Boundary

The client sends selected IDs and explicit edits.

The server computes:

- current affected entities
- expected versions
- lifecycle eligibility
- consequence preview
- reversibility
- protection/deadline warnings

The client may display consequences but may not submit altered consequences as authority.

### 9.3 Submit Confirmation

```txt
POST /action-confirmations/{confirmationId}/confirm
```

Request:

```txt
ConfirmActionRequest
- confirmationVersion
- idempotencyKey
- acknowledgedWarningCodes[]
```

The server:

```txt
reloads current state
→ verifies exact confirmation version
→ revalidates consequences
→ executes deterministic command
→ writes audit/outbox
→ returns CommandResult
```

A stale or invalid confirmation never executes partially.

### 9.4 Confirmation Expiration

On expiration or invalidation:

```txt
no mutation
→ old confirmation cannot be revived
→ client requests a fresh confirmation preview
→ user confirms again
```

---

## 10. Command Result Contract

```txt
CommandResultView
- commandId
- status
- commandType
- idempotencyKeyReference
- committedAt?
- affectedEntities[]
- sourceConfirmationId
- conflict?
- failure?
- noOp
```

Statuses:

```txt
SUCCEEDED
CONFLICTED
FAILED_RETRYABLE
FAILED_FINAL
CANCELLED_BEFORE_EXECUTION
```

### 10.1 No-Op Success

Matching already-applied terminal state may return:

```txt
status = SUCCEEDED
noOp = true
```

It must not create duplicate events or version changes.

### 10.2 Affected Entity Result

```txt
AffectedEntityResult
- entityType
- entityId
- previousVersion?
- currentVersion
- currentStatus
- operation
```

For newly created entities, canonical IDs appear only here after commit.

---

## 11. Expected Version Contract

Expected versions are required for:

- direct edits to canonical entities
- terminal lifecycle actions
- Restore
- Routine Stop
- continuation creation where source version matters
- bulk mutations
- parent terminal finalization

Expected versions may be carried inside the server-generated confirmation rather than repeated manually by the client.

The final command must use the version snapshot bound to the confirmation.

The client cannot replace it with a newer version without generating a new confirmation.

---

## 12. Idempotency Contract

Every consequential command submission requires an idempotency key.

Recommended transport:

```txt
Idempotency-Key header
```

Rules:

- same key + same request hash returns prior result
- same key + different request hash returns conflict
- retry after lost response uses the same key
- UI must retain the key until a definitive result is known
- refreshing the page must not silently generate a new key for an in-flight confirmation
- a new explicit confirmation gets a new key

Starting AI attempts may also use optional idempotency keys to prevent duplicate attempts after lost responses, but AI attempt idempotency is not equivalent to mutation idempotency.

---

## 13. Error Envelope

```txt
ApiError
- code
- category
- message
- userMessageKey
- retryable
- refreshRequired
- fieldErrors[]
- conflictContext?
- operationReference?
```

Categories:

```txt
VALIDATION
AUTHENTICATION
AUTHORIZATION
NOT_FOUND
CONFLICT
POLICY_BLOCKED
AI_UNAVAILABLE
AI_OUTPUT_REJECTED
RATE_LIMITED
TIMEOUT
CANCELLED
INTERNAL
```

### 13.1 Conflict Context

```txt
ConflictContext
- conflictType
- entityId?
- expectedVersion?
- currentVersion?
- currentStatus?
- changedMaterialFields[]
- sourceExpired
- refreshedResourceUrl?
```

Conflict codes include:

```txt
CONFLICT_STALE_VERSION
CONFLICT_CONFIRMATION_INVALIDATED
CONFLICT_DRAFT_SUPERSEDED
CONFLICT_PARENT_TERMINAL
CONFLICT_IDEMPOTENCY_HASH_MISMATCH
CONFLICT_SELECTION_CHANGED
```

### 13.2 Security Boundary

Errors must not expose:

- provider raw messages
- prompt content
- stack traces
- cross-user entity existence
- crisis classification details beyond approved user-facing safety flow
- internal rule implementation details that enable bypass

---

## 14. Refresh Contract After Conflict

On stale conflict:

```txt
1. preserve user-authored input and edits locally
2. fetch current authoritative source resource
3. request regenerated preview
4. show material changes
5. require explicit confirmation again
```

The client must not automatically resubmit the old confirmation.

The server may return a `refreshedResourceUrl`, but must not silently create a new confirmed action.

For PlanningDraft edits, rebase is explicit:

```txt
client submits old edits against new base revision
→ server validates each edit
→ incompatible edits are returned as field conflicts
```

Automatic semantic merging by AI is forbidden in MVP.

---

## 15. Bulk Action Contract

Bulk requests contain explicit IDs.

```txt
BulkActionSelection
- selectedIds[]
- actionType
```

The confirmation resource binds:

- exact selected IDs
- exact expected versions
- exact consequence set
- warning acknowledgements

Execution is all-or-nothing.

If one item becomes stale or invalid:

```txt
status = CONFLICTED
no item mutates
refreshRequired = true
```

The response may identify conflicting items, but must not imply that valid items were applied.

---

## 16. Frontend Planning State Machine

```txt
IDLE
→ SUBMITTING
→ QUEUED | RUNNING
→ REVIEWABLE
→ EDITING
→ CONFIRMATION_LOADING
→ AWAITING_CONFIRMATION
→ COMMAND_SUBMITTING
→ SUCCEEDED | CONFLICTED | FAILED
```

Alternative terminal branches:

```txt
QUEUED | RUNNING
→ CANCELLING
→ CANCELLED

QUEUED | RUNNING
→ AI_FAILED
→ MANUAL_FALLBACK
```

Rules:

- no `PARTIAL_REVIEWABLE` state
- navigation does not imply server cancellation unless cancellation is submitted
- user input persists through AI failure
- draft edits persist locally until server revision creation succeeds
- command success is never inferred from HTTP connection closure

---

## 17. Frontend Reconcile State Machine

```txt
IDLE
→ STARTING
→ COMPUTING_FACTS
→ FACTS_READY
→ EXPLANATION_PENDING
→ COMPLETE
```

Degraded branch:

```txt
FACTS_READY
+ explanation failure
→ DEGRADED_FACTS_ONLY
```

Failure branch:

```txt
COMPUTING_FACTS
→ RECONCILE_UNAVAILABLE
```

Recommendation action branch:

```txt
RECOMMENDATION_SELECTED
→ CONFIRMATION_LOADING
→ AWAITING_CONFIRMATION
→ COMMAND_SUBMITTING
→ APPLIED | ACCEPTED_BUT_CONFLICTED | FAILED
```

The UI must not collapse these into one “accepted” state.

---

## 18. User Input Preservation

The frontend preserves user-authored input separately from server AI results.

At minimum:

- original planning text
- unsaved draft edits
- selected recommendation IDs
- warning acknowledgements before submission
- in-flight idempotency key

Provider failure, timeout, cancellation, or conflict must not erase this input.

Sensitive user text must not be placed in analytics events or error-reporting breadcrumbs.

Local persistence duration and encryption strategy are deferred to implementation planning.

---

## 19. Retry Contract

### 19.1 Safe Client Retries

Clients may safely retry:

- GET resource requests
- cancelled-state retrieval
- same-key command submission after unknown transport result
- AI attempt creation only with the same attempt idempotency identity

### 19.2 Unsafe Automatic Retries

Clients must not automatically retry with a new identity:

- confirmed mutations
- bulk mutations
- cancellation after attempt already completed
- stale confirmation
- policy-blocked requests

### 19.3 Retry Presentation

The API exposes:

```txt
retryable
retryAfter?
manualFallbackAvailable
refreshRequired
```

The frontend should not infer retryability solely from HTTP status.

---

## 20. HTTP Status Mapping Proposal

Suggested mapping:

```txt
200 success or idempotent replay result
201 resource created
202 async attempt accepted
400 malformed request
401 unauthenticated
403 unauthorized or policy-blocked where disclosure is safe
404 resource absent within user scope
409 stale version, invalid confirmation, idempotency mismatch
422 semantically invalid user input or draft edit
429 rate limited
499-equivalent not standardized; use application CANCELLED state
500 internal failure
502/503 AI/provider unavailable where applicable
504 timeout
```

Application error code remains authoritative; clients must not depend only on HTTP status.

---

## 21. Polling Contract

Polling responses include:

```txt
status
resourceVersion
recommendedPollAfterMs
terminal
```

Rules:

- clients use bounded backoff
- server may advise slower polling
- polling stops on terminal state
- polling after cancellation returns durable cancelled state
- polling must not create new AI attempts
- ETag or version-based conditional GET may be used

Exact intervals belong to 020C or implementation configuration.

---

## 22. Authorization and Ownership

Every resource lookup is scoped to the authenticated user/workspace.

The API must not reveal whether another user’s resource ID exists.

Server-side checks apply to:

- attempt retrieval
- draft retrieval/edit
- Reconcile session retrieval
- confirmation creation and submission
- command result retrieval
- cancellation

Client-provided `userId` is never authority when identity is available from authentication context.

---

## 23. Scenario Checks

### Scenario A — Planning completes quickly

- POST returns `COMPLETED`
- draft is immediately available
- client enters REVIEWABLE

### Scenario B — Planning takes longer

- POST returns `202` with `RUNNING`
- client polls
- no partial draft is visible
- validated draft appears only at completion

### Scenario C — User cancels, provider responds late

- cancellation becomes durable
- late output is discarded
- polling continues to return CANCELLED
- no draft revision appears

### Scenario D — User edits a draft while canonical context changes

- edit revision may be created
- confirmation eligibility becomes stale
- confirmation request returns refreshed consequences or conflict
- old preview cannot execute

### Scenario E — Recommendation accepted but command conflicts

- recommendation outcome records ACCEPTED
- CommandResult is CONFLICTED
- UI displays accepted but not applied
- refreshed confirmation is required

### Scenario F — Bulk Drop contains one newly protected Task

- confirmation is invalidated or command conflicts
- no Tasks are dropped
- conflicting Task is identified safely

### Scenario G — Response lost after commit

- client retries same confirmation with same idempotency key
- server returns existing successful CommandResult
- no duplicate event or mutation

### Scenario H — Reconcile AI explanation fails

- deterministic facts and rule matches remain available
- session enters DEGRADED_FACTS_ONLY
- no free-form replacement summary is generated

### Scenario I — Deterministic Reconcile fails

- session becomes FAILED
- facts are unavailable
- AI explanation is not requested
- manual Today/execution remains available

### Scenario J — User refreshes during command submission

- client restores in-flight idempotency key
- retrieves or retries same command identity
- does not create duplicate confirmation or command

---

## 24. Stabilized Proposals Pending Claude Review

- Planning and Reconcile use resource creation plus polling in MVP.
- token-level streaming is disabled.
- no approvable partial AI state exists.
- PlanningDraft revisions are immutable.
- server-generated confirmation resources bind exact previews and versions.
- confirmed mutations require idempotency keys.
- client-provided consequences are never authoritative.
- conflicts preserve user input and require refreshed confirmation.
- bulk commands are all-or-nothing.
- Reconcile facts can remain usable without AI explanation.
- deterministic Reconcile failure disables Reconcile rather than invoking AI over raw data.
- cancellation is durable and late output is discarded.
- recommendation acceptance and command success remain separate states.

---

## 25. Open Questions for Claude

### Q1. Are the proposed resource boundaries correct?

Review whether PlanningAttempt, PlanningDraft, ReconcileSession, ActionConfirmation, and CommandResult are sufficient without creating a generic WorkflowRun abstraction.

### Q2. Is polling the correct MVP transport?

Review whether any user-facing need justifies SSE before validated completion.

### Q3. Should PlanningAttempt and PlanningDraft remain separate resources?

Current proposal: yes, because an attempt may fail or be cancelled without producing a draft.

### Q4. Is immutable draft revisioning proportional for MVP?

Review whether revisions should be full snapshots or patch chains. Current proposal: full snapshot persistence from 019C.

### Q5. Is the confirmation resource model coherent?

Review server authority, exact revision binding, expiry, warning acknowledgement, and stale invalidation.

### Q6. Are expected versions sufficiently protected from client substitution?

Current proposal: bind them inside the server-generated confirmation and reject altered or outdated confirmation versions.

### Q7. Is the idempotency contract complete for page refresh and lost responses?

Review key persistence, request hashing, same-key replay, and new confirmation identity.

### Q8. Does the error envelope distinguish all meaningful failure classes?

Focus on provider failure, output rejection, deterministic Reconcile failure, conflict, cancellation, and policy block.

### Q9. Is `CONFIRMED` on PlanningDraft potentially misleading?

Review whether draft status should instead remain REVIEWABLE/SUPERSEDED and confirmation linkage be represented separately.

### Q10. Is user-input preservation sufficiently privacy-safe?

Review local persistence, analytics exclusion, and error-reporting boundaries without expanding into implementation-specific encryption.

### Q11. Should cancellation itself require an idempotency key?

Current proposal: cancellation is naturally idempotent through attempt state and expected version.

### Q12. Are accepted-but-conflicted recommendations represented clearly enough?

Review API fields and frontend state naming.

### Q13. Should command submission return synchronously after database commit?

Current proposal: yes. Long AI generation is asynchronous; deterministic mutations remain short synchronous transactions.

### Q14. Is `422` appropriate for semantically invalid draft edits?

Review consistency with `409` for stale revision and `400` for malformed transport.

### Q15. Is explicit rebase of user edits enough after conflict?

Current proposal rejects automatic AI semantic merging.

### Q16. Should warnings acknowledgements be codes only?

Review whether warning instances need IDs/version binding to prevent acknowledging a changed warning using an old code.

### Q17. Are polling resource versions and ETags sufficient for reconnect behavior?

Review duplicate attempt creation and terminal-state retrieval.

### Q18. Are any Discussion 018A confirmation guarantees lost at the API boundary?

Focus on exact selection, consequences, reversibility, protection, deadline context, and revalidation.

---

## 26. Mind Map Impact — Proposed, Do Not Apply Yet

### Product Vision

- AI work is reviewable before any canonical change.
- retries, refreshes, and weak networks do not duplicate consequences.
- user input survives technical failure.

### MVP Core Loop

```txt
user input
→ create AI attempt
→ retrieve validated resource
→ review/edit
→ request authoritative confirmation
→ explicit confirm
→ deterministic command
→ authoritative result
```

### User Flow

Add:

- running attempt state
- cancellable attempt state
- reviewable draft revision
- facts-only Reconcile degraded mode
- stale confirmation refresh
- accepted-but-not-applied state
- same-command recovery after lost response

### AI Responsibilities

- AI produces proposal or explanation resources only.
- AI output never becomes mutation authority.
- no partial output reaches a reviewable state.

### AI Guardrails

- no client-authored consequence authority
- no stale confirmation execution
- no duplicate command after refresh
- no automatic semantic merge after conflict
- no AI fallback when deterministic Reconcile fails

### Data Events

Candidates for 019C alignment:

```txt
PLANNING_ATTEMPT_STARTED
PLANNING_ATTEMPT_CANCELLED
PLANNING_DRAFT_REVISION_CREATED
CONFIRMATION_CREATED
CONFIRMATION_INVALIDATED
COMMAND_RESULT_RETURNED
RECONCILE_DEGRADED_MODE_ENTERED
USER_INPUT_RESTORED_AFTER_FAILURE
```

### Traction and Guardrail Metrics

- attempt completion rate
- attempt cancellation rate
- stale confirmation rate
- accepted-but-conflicted rate
- idempotent replay rate
- draft edit conflict rate
- partial-reviewable state count, expected zero
- duplicate command count, expected zero

### Current Decisions

Proposed:

- resource-oriented async AI attempts
- polling transport
- immutable draft revisions
- server-generated confirmation resources
- synchronous deterministic command completion
- explicit conflict refresh

### Open Questions

- final resource naming
- confirmation status naming
- warning acknowledgement identity
- polling backoff defaults

---

## 27. Affected Formal Documents — Record Only

After closure, update or create:

- API resource specification
- OpenAPI contract
- frontend Planning state machine
- frontend Reconcile state machine
- confirmation and command submission contract
- error-code catalog
- idempotency client-handling guide
- conflict refresh and draft-rebase specification
- contract and integration tests in Discussion 021
- implementation sequencing in Discussion 022

Potential ADRs:

- polling rather than token streaming for validated AI resources
- server-generated confirmation resource
- separation of proposal acceptance from command success

---

## 28. Structured Review Request for Claude

Review this exact proposal for API coherence, state-machine completeness, retry safety, concurrency correctness, privacy, and proportional MVP complexity.

For each finding provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected section or contract
4. concrete client/server failure scenario
5. why the proposal is ambiguous, unsafe, contradictory, or incomplete
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- attempt versus draft resource identity
- polling and cancellation races
- immutable draft revisions
- confirmation expiry and invalidation
- expected-version binding
- idempotency after refresh or lost response
- error envelope ambiguity
- accepted-but-conflicted representation
- bulk all-or-nothing semantics
- user-input preservation
- warning acknowledgement identity
- stale edit rebase
- deterministic Reconcile failure
- frontend state completeness

Do not expand into provider selection, prompt content, retry threshold values, UI styling, authentication vendor choice, offline-first architecture, or implementation sequencing.

This discussion remains open until Claude reviews the proposal and Mahdi resolves accepted findings.
