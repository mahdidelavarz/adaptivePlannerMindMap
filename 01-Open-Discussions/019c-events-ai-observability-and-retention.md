# Discussion 019C — Events, AI Observability, and Retention

## Status

Open for GPT × Claude review.

This document is Part C of the Discussion 019 persistence program.

Companion discussions:

- [[01-Open-Discussions/019-data-model-and-ai-observability]]
- [[01-Open-Discussions/019a-final-canonical-data-model-resolution]]
- [[01-Open-Discussions/019b-final-transactions-concurrency-and-idempotency-resolution]]

Accepted dependencies:

- [[01-Open-Discussions/012-core-product-model]]
- [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]
- [[01-Open-Discussions/014-ai-planning-output-contract]]
- [[01-Open-Discussions/015-task-and-routine-execution-model]]
- [[01-Open-Discussions/016-reconcile-trigger-and-severity]]
- [[01-Open-Discussions/017b-final-reconcile-intelligence-resolution]]
- [[01-Open-Discussions/018a-final-action-permission-trust-and-reversibility-resolution]]
- [[01-Open-Discussions/018c-final-failure-privacy-domain-and-hostile-input-resolution]]
- [[01-Open-Discussions/019a-final-canonical-data-model-resolution]]
- [[01-Open-Discussions/019b-final-transactions-concurrency-and-idempotency-resolution]]

---

## 1. Scope

This discussion defines the minimum honest event and observability model required to:

- preserve an auditable history of consequential product actions
- evaluate Planning AI and Reconcile AI without confusing generated text with product truth
- represent PlanningDraft review and suggestion outcomes
- represent Reconcile sessions, rule matches, groups, recommendations, and user decisions
- correlate commands, transactions, proposals, confirmations, and events
- retain provider-neutral model and prompt-template metadata
- distinguish product events, audit records, AI operational diagnostics, and raw content
- support privacy deletion and retention boundaries
- observe degraded mode, manual fallback, hostile content, and crisis safety flows
- compute analytics without persisting misleading Goal progress or psychological inference

This discussion does not reopen canonical entity semantics or transaction rules accepted in 019A and 019B.

---

## 2. Explicitly Out of Scope

This discussion does not finalize:

- physical SQL syntax for one database
- exact retention durations in days
- jurisdiction-specific legal requirements
- provider contract terms
- encryption implementation
- queue technology
- warehouse or BI vendor
- endpoint DTOs
- exact dashboard layouts
- final crisis-response copy
- model-evaluation thresholds

Those belong to Discussions 020–022 and legal/security review.

---

## 3. Governing Observability Principle

Observability must explain what happened without becoming a second mutable source of domain truth.

```txt
canonical domain state
→ authoritative current truth

domain/audit events
→ durable historical facts about accepted transitions

AI operational records
→ diagnostics about model-assisted attempts

raw prompt/response content
→ sensitive optional debug material, not ordinary analytics
```

The system must never derive a product mutation merely because an event or AI log claims it occurred.

The canonical tables remain authoritative for current state.

---

## 4. Four Persistence Layers

The system must distinguish four layers:

```txt
1. CANONICAL_DOMAIN
2. DOMAIN_EVENT_AND_AUDIT
3. AI_OPERATIONAL_OBSERVABILITY
4. RAW_AI_CONTENT
```

### 4.1 CANONICAL_DOMAIN

Contains current accepted Goal, Project, Task, Routine, RoutineOccurrence, and temporary PlanningDraft state defined elsewhere.

### 4.2 DOMAIN_EVENT_AND_AUDIT

Contains durable facts about:

- user-confirmed actions
- deterministic system transitions
- previous and resulting state references
- rule/version context where applicable
- confirmation and consequence-preview linkage
- transaction and idempotency linkage

### 4.3 AI_OPERATIONAL_OBSERVABILITY

Contains minimal diagnostics about an AI attempt:

- request identity
- feature surface
- provider-neutral model profile
- prompt-template version
- latency and token usage
- validation result
- failure category
- degraded/fallback outcome

### 4.4 RAW_AI_CONTENT

May contain prompt or response bodies only under restricted, separately controlled retention.

Raw content must not be required for ordinary product analytics or audit correctness.

---

## 5. Event Envelope

Every durable domain or audit event uses a provider-neutral envelope.

Proposed minimum:

```txt
EventEnvelope
- eventId
- userId
- eventType
- eventVersion
- occurredAt
- actorType: USER | SYSTEM_DETERMINISTIC
- actorId?
- aggregateType
- aggregateId
- aggregateVersion?
- commandId?
- transactionId
- correlationId
- causationId?
- idempotencyKeyHash?
- proposalId?
- confirmationId?
- ruleId?
- ruleVersion?
- payload
- payloadSchemaVersion
- createdAt
```

Rules:

- `eventId` is globally unique.
- `eventVersion` versions the semantic meaning of the event type.
- `payloadSchemaVersion` versions the payload shape.
- `transactionId` groups events committed in one canonical transaction.
- `correlationId` groups one user-visible flow across multiple transactions.
- `causationId` points to the immediate command/event/proposal that caused this event.
- AI is never recorded as the authorizing actor.
- model/provider identifiers do not belong in ordinary domain event payloads unless required to explain an AI proposal reference.

---

## 6. Actor and Authority Boundary

Allowed authorizing actors:

```txt
USER
SYSTEM_DETERMINISTIC
```

`SYSTEM_DETERMINISTIC` applies only to transitions already accepted by product rules, such as:

- automatic MISSED marking
- REVIEW_DUE derivation/materialization where persisted
- Project-owned Routine shutdown during accepted Project terminal transition
- deterministic default application

AI may be recorded as proposal provenance, never as mutation authority.

Incorrect:

```txt
actorType = AI
```

Correct:

```txt
actorType = USER
proposalId = aiProposalId
```

or:

```txt
actorType = SYSTEM_DETERMINISTIC
causationId = confirmedProjectTransition
```

---

## 7. PlanningDraft Persistence

A PlanningDraft is not a canonical Plan entity.

For MVP, PlanningDraft may be temporarily persisted to support:

- crash recovery
- multi-device review
- retry after provider failure
- stale-context detection
- proposal/version comparison
- review before approval

Proposed model:

```txt
PlanningDraft
- id
- userId
- status: ACTIVE | APPROVED | REJECTED | EXPIRED | INVALIDATED
- sourceRequestId
- currentRevision
- domainClassification?
- createdAt
- updatedAt
- expiresAt
- approvedAt?
- rejectedAt?
- invalidatedAt?
- invalidationReasonCode?
```

Draft content should be stored as versioned revisions rather than silently overwritten.

```txt
PlanningDraftRevision
- id
- planningDraftId
- revisionNumber
- schemaVersion
- structuredDraftPayload
- generatedByOperationId?
- validationStatus: VALID | INVALID
- validationReasonCodes
- createdAt
```

Rules:

- no canonical entity is created until approval transaction commits
- only one revision is current
- prior revisions remain available for short-lived audit/review until retention expires
- rejected or expired drafts do not become domain history equivalent to created entities
- draft retention inherits the sensitive-data boundary from 018C
- raw conversation text should not be duplicated into every revision

---

## 8. Planning Proposal and Suggestion Outcomes

The product must distinguish generation from user decision.

Proposed records:

```txt
AIProposal
- id
- userId
- feature: PLANNING | RECONCILE
- operationId
- planningDraftId?
- reconcileSessionId?
- proposalVersion
- proposalSchemaVersion
- status: GENERATED | VALIDATED | REJECTED_INVALID | PRESENTED | SUPERSEDED | EXPIRED
- createdAt
```

```txt
ProposalItem
- id
- proposalId
- itemType
- clientStableKey
- affectedEntityId?
- proposedActionType?
- proposedStructuredValue?
- consequencePreviewVersion?
- reversible?
```

```txt
ProposalItemDecision
- id
- proposalItemId
- decision: ACCEPTED | ACCEPTED_EDITED | REJECTED | CANCELLED
- resultingEntityId?
- resultingCommandId?
- decidedAt
```

`IGNORED` should not be persisted as a semantic decision merely because no action occurred.

Suggested rule:

```txt
no explicit decision before proposal expiry
→ proposal outcome may be EXPIRED_WITHOUT_DECISION
→ individual items are not labeled REJECTED
```

This prevents absence from being misrepresented as dislike or rejection.

---

## 9. Approval and Confirmation Linkage

Every consequential AI-assisted mutation must be traceable through:

```txt
AI operation
→ proposal
→ presented preview
→ explicit confirmation
→ command
→ canonical transaction
→ domain/audit events
```

Proposed confirmation record:

```txt
ActionConfirmation
- id
- userId
- proposalId?
- actionType
- selectedEntityIds
- expectedEntityVersions
- consequencePreviewVersion
- previewHash
- confirmedAt
- invalidatedAt?
- invalidationReasonCode?
- committedCommandId?
```

Rules:

- confirmation is not itself proof that mutation committed
- event/command result determines whether mutation committed
- stale confirmation remains historical but cannot be reused
- bulk selected set must be preserved exactly
- confirmation content should use IDs and hashes rather than duplicating sensitive titles/descriptions

---

## 10. ReconcileSession Decision

A persisted `ReconcileSession` is required for MVP.

Reason:

- one session may span multiple rule groups and user decisions
- some decisions commit in separate transactions
- deterministic facts may remain usable when AI explanation fails
- event-only derivation would make session boundaries ambiguous

Proposed model:

```txt
ReconcileSession
- id
- userId
- status: OPEN | COMPLETED | ABANDONED | EXPIRED
- triggerType: USER_OPENED | SYSTEM_PROMPTED | REVIEW_CHECKPOINT
- openedAt
- completedAt?
- abandonedAt?
- ruleCatalogVersion
- factsSnapshotVersion
- aiExplanationMode: ENABLED | DEGRADED | NOT_REQUESTED
- correlationId
```

A session is organizational state, not canonical execution truth.

---

## 11. Reconcile Facts, Matches, and Groups

Structured deterministic facts must be representable without free text.

```txt
ReconcileFact
- id
- sessionId
- entityType
- entityId
- factType
- metricName?
- numericValue?
- dateValue?
- booleanValue?
- reasonCode
- evidenceQuality
- observedAt
```

```txt
RuleMatch
- id
- sessionId
- ruleId
- ruleVersion
- severity
- matchedAt
- status: ACTIVE | RESOLVED | DISMISSED | EXPIRED
```

```txt
RuleMatchEntity
- ruleMatchId
- entityType
- entityId
- role: PRIMARY | RELATED | PARENT | CHILD
```

```txt
ReconcileGroup
- id
- sessionId
- groupType
- ownerEntityType?
- ownerEntityId?
- ruleMatchId?
- displayOrderDerivedKey?
```

```txt
ReconcileGroupItem
- groupId
- entityType
- entityId
- reasonCode
```

Rules:

- free-text notes and descriptions are not persisted into Reconcile fact records
- metrics are accepted structured observations, not AI inference
- group membership uses canonical IDs
- display ordering is not canonical Task ordering
- evidence quality must not become fabricated probability

---

## 12. Reconcile Recommendation Outcomes

A recommendation is not a mutation.

```txt
ReconcileRecommendation
- id
- sessionId
- ruleMatchId
- proposalId?
- actionType
- affectedEntityIds
- recommendationVersion
- presentationClass: RULE_ATTACHED_ACTION | FIXED_SYSTEM_OPTION
- presentedAt?
- outcome: PENDING | ACCEPTED | ACCEPTED_EDITED | REJECTED | CANCELLED | EXPIRED
- decidedAt?
- resultingCommandId?
```

Rules:

- `ACCEPTED` means the user chose the recommendation, not that the mutation necessarily committed
- committed result must be linked separately
- `REJECTED` requires an explicit rejection action
- closing the session does not automatically reject every remaining recommendation
- `EXPIRED` or `CANCELLED` is preferable to inventing negative preference
- AI explanation text is not required to evaluate recommendation acceptance

---

## 13. AI Operation Record

Every model request creates an operational record even when no proposal is produced.

```txt
AIOperation
- id
- userId
- feature: PLANNING | RECONCILE_EXPLANATION | DOMAIN_CLASSIFICATION | SAFETY_CLASSIFICATION
- status: STARTED | SUCCEEDED | FAILED | REFUSED | REJECTED_OUTPUT | CANCELLED
- providerProfileId
- modelProfileId
- promptTemplateId
- promptTemplateVersion
- outputSchemaVersion?
- requestCorrelationId
- startedAt
- completedAt?
- latencyMs?
- inputTokenCount?
- outputTokenCount?
- retryAttempt
- failureCategory?
- validationResult?
- fallbackMode?
- rawContentRecordId?
```

The domain model must not depend on vendor-specific provider names.

Provider/model profiles may map internal stable IDs to runtime configuration.

---

## 14. Provider-Neutral Versioning

Proposed metadata concepts:

```txt
ProviderProfile
- stable internal ID
- runtime provider family
- configuration version

ModelProfile
- stable internal ID
- capability class
- runtime model identifier
- configuration version

PromptTemplate
- stable template ID
- feature
- semantic version
- content hash

RuleCatalog
- catalog version
- activatedAt
```

Domain entities should store only source provenance such as `MANUAL` or `AI_ASSISTED`.

Provider/model/prompt details belong to AI operational records and proposal linkage, not Goal/Task/Routine rows.

---

## 15. Failure and Degraded-Mode Observability

Flow-affecting failures must be visible to the user and observable internally.

Minimum failure categories inherited from 018C:

```txt
PROVIDER_UNAVAILABLE
TIMEOUT
RATE_LIMITED
SAFETY_REFUSAL
MALFORMED_OUTPUT
SCHEMA_INVALID
PARTIAL_OUTPUT
STALE_CONTEXT
RULE_ENGINE_MISMATCH
AUTHORIZATION_MISMATCH
INTERNAL_PROCESSING_ERROR
```

Suggested events/records:

```txt
AI_REQUEST_FAILED
AI_OUTPUT_REJECTED
AI_PARTIAL_OUTPUT_REJECTED
AI_SCHEMA_VALIDATION_FAILED
AI_DEGRADED_MODE_ENTERED
MANUAL_FALLBACK_USED
```

User-facing copy is not stored in ordinary event payloads unless a compliance requirement later demands it.

The system may store a message-template code instead.

---

## 16. Hostile Content and Prompt-Injection Observability

The system should observe security-relevant handling without copying hostile content into broad analytics.

Proposed events:

```txt
HOSTILE_CONTENT_DETECTED
PROMPT_INJECTION_BLOCKED
UNAUTHORIZED_ACTION_NAME_BLOCKED
SECRET_PATTERN_REDACTED
```

Minimum payload:

- sourceType: TASK | GOAL | PROJECT | IMPORTED_EMAIL | CALENDAR | DOCUMENT | OTHER
- detectorVersion
- detectionCategory
- actionBlocked
- featureSurface
- occurredAt

Do not include the full hostile text in ordinary events.

A restricted raw-content reference may be retained only when debug/security policy permits.

Detection does not change canonical user content or imply user intent.

---

## 17. Crisis Safety Observability

Crisis observability must support the Discussion 021 release gate without creating a durable psychological profile.

Proposed events:

```txt
CRISIS_SAFETY_FLOW_TRIGGERED
CRISIS_RESOURCE_PRESENTED
CRISIS_LOCATION_UNAVAILABLE
CRISIS_AI_DRAFT_BLOCKED
CRISIS_RELEASE_GATE_EVALUATED
```

Rules:

- do not persist detailed crisis narrative in ordinary analytics
- do not derive durable labels such as `userIsHighRisk`
- store operation-scoped safety category only where required
- resource locale/country code may be recorded without precise location
- crisis draft generation rate must be measurable and expected to remain zero
- access to crisis-related observability must be restricted
- retention requires dedicated safety/legal review

---

## 18. RoutineOccurrence Exposure and Invalidation Events

019B defines an occurrence as exposed only when returned in a user-visible execution response.

Proposed event:

```txt
ROUTINE_OCCURRENCE_EXPOSED
- occurrenceId
- surface: TODAY | EXECUTION_DAY | ROUTINE_DETAIL | OTHER
- responseId
- exposedAt
```

This event should be deduplicated semantically for first exposure where the deletion/invalidation policy depends on whether exposure ever occurred.

Invalid future materialization events:

```txt
ROUTINE_OCCURRENCE_INVALIDATED
- occurrenceId
- routineId
- reasonCode: OUTSIDE_FINAL_EFFECTIVE_RANGE
- invalidatedAt
```

Open design decision:

- derive first exposure from immutable exposure events, or
- persist `firstExposedAt` on RoutineOccurrence as an operational projection

The event remains historical; a projection may exist for efficient transaction checks.

---

## 19. Domain Event Taxonomy

Minimum lifecycle events:

### Goal

```txt
GOAL_CREATED
GOAL_UPDATED
GOAL_CONTINUATION_CONFIRMED
GOAL_REVIEW_DEFERRED
GOAL_ACHIEVED
GOAL_ABANDONED
```

### Project

```txt
PROJECT_CREATED
PROJECT_UPDATED
PROJECT_COMPLETED
PROJECT_STOPPED
PROJECT_TERMINAL_FINALIZATION_BLOCKED
```

### Task

```txt
TASK_CREATED
TASK_UPDATED
TASK_COMPLETED
TASK_DROPPED
TASK_RESTORED
TASK_MOVED_TO_BACKLOG
TASK_REPLANNED
TASK_SPLIT
```

### Routine

```txt
ROUTINE_CREATED
ROUTINE_UPDATED
ROUTINE_STOPPED
ROUTINE_CONTINUATION_CREATED
```

### RoutineOccurrence

```txt
ROUTINE_OCCURRENCE_CREATED
ROUTINE_OCCURRENCE_EXPOSED
ROUTINE_OCCURRENCE_DONE
ROUTINE_OCCURRENCE_MISSED
ROUTINE_OCCURRENCE_CORRECTED
ROUTINE_OCCURRENCE_INVALIDATED
```

### Ownership and structure

```txt
ENTITY_ATTACHED
ENTITY_DETACHED
ENTITY_REPARENTED
PARENT_EMPTY_CONSEQUENCE_RECORDED
```

Events should describe accepted facts, not UI clicks that did not alter or meaningfully decide anything.

---

## 20. Command and Transaction Events

Minimum operational/audit events from 019B:

```txt
COMMAND_RECEIVED
COMMAND_CONFLICTED
COMMAND_IDEMPOTENT_REPLAYED
COMMAND_NOOP_SUCCEEDED
TRANSACTION_COMMITTED
TRANSACTION_ROLLED_BACK
CASCADE_COMPLETED
BULK_ACTION_REJECTED
OCCURRENCE_DUPLICATE_PREVENTED
OUTBOX_DELIVERY_RETRIED
```

Not every command diagnostic needs indefinite retention.

Lifecycle events and authoritative audit decisions generally outlive low-level transaction diagnostics.

---

## 21. Event Granularity for Cascades

Project terminal cascades should produce both:

1. one parent semantic event
2. child lifecycle events for each changed Routine

Example:

```txt
PROJECT_COMPLETED
ROUTINE_STOPPED × N
CASCADE_COMPLETED
```

All share the same `transactionId` and `correlationId`.

Reason:

- parent event explains user intent
- child events preserve each Routine's lifecycle history
- cascade event supports operational completeness checks

A single giant payload containing every child is not sufficient as the only history representation.

---

## 22. Outbox Record

019B requires durable event intent in the canonical transaction.

Proposed outbox minimum:

```txt
OutboxRecord
- id
- transactionId
- eventId
- eventType
- eventVersion
- payload
- createdAt
- publishedAt?
- attemptCount
- nextAttemptAt?
- lastErrorCode?
```

Rules:

- `eventId` unique prevents semantic duplicate publication
- publisher retries do not create a new domain event
- broker delivery is not canonical truth
- retention of delivered outbox rows may be shorter than retention of audit/domain events

---

## 23. Correlation and Causation Rules

### Correlation ID

One user-visible flow may contain several transactions.

Examples:

- staged Goal terminal resolution
- staged Project terminal resolution
- multi-step PlanningDraft approval
- Reconcile session with several accepted actions

All use one `correlationId`.

### Causation ID

Immediate cause examples:

```txt
AIProposal → ActionConfirmation
ActionConfirmation → Command
Command → TASK_DROPPED
PROJECT_COMPLETED → ROUTINE_STOPPED
```

Causation is directional and must not be inferred solely from timestamp proximity.

---

## 24. Retention Classes

Exact durations are deferred, but semantic classes must be accepted now.

```txt
R1 — CANONICAL_AND_AUDIT_HISTORY
R2 — PRODUCT_DECISION_OBSERVABILITY
R3 — AI_OPERATIONAL_DIAGNOSTICS
R4 — RAW_AI_CONTENT
R5 — SECURITY_AND_CRISIS_RESTRICTED
R6 — EPHEMERAL_DRAFT_CONTENT
```

### R1 — Canonical and Audit History

Examples:

- lifecycle events
- explicit confirmations
- Restore/Drop history
- rule/version linkage for accepted actions

Long-lived subject to account deletion/legal policy.

### R2 — Product Decision Observability

Examples:

- proposal presented
- explicit accept/edit/reject
- Reconcile session outcome
- fallback used

May use bounded retention or aggregation after detailed records expire.

### R3 — AI Operational Diagnostics

Examples:

- latency
- token counts
- model profile
- failure category
- schema validation result

Shorter retention than audit history.

### R4 — Raw AI Content

Default:

- not retained indefinitely
- excluded from ordinary analytics
- access restricted
- separately enabled for debugging
- redacted where feasible
- deletable independently from derived non-sensitive operational metadata where legally permitted

### R5 — Security and Crisis Restricted

Strict access, dedicated retention review, no ordinary analytics joins that create user profiles.

### R6 — Ephemeral Draft Content

PlanningDraft revisions and unapproved structured content expire after a bounded review window unless an active user flow requires them.

---

## 25. Redaction and Forbidden Persistence

The following must never be intentionally persisted in AI logs:

- passwords
- authentication tokens
- API keys
- payment credentials
- private cryptographic material

The following are excluded from ordinary AI observability by default:

- unrelated notes
- entire account history
- imported private communications
- precise location
- detailed health/legal/financial narratives
- full hostile prompt-injection text
- full crisis narrative

Forbidden derived persistence:

- motivation score
- discipline score
- mental-health likelihood
- personality profile
- inferred capacity score
- Goal-achievement probability
- universal progress percentage without accepted semantic basis

---

## 26. Privacy Deletion and Referential Integrity

Privacy deletion must not leave raw sensitive content reachable through observability references.

Proposed principles:

- delete or irreversibly anonymize raw prompt/response content according to account deletion policy
- remove user-authored draft content
- preserve only legally permitted minimal aggregate/non-identifying diagnostics
- avoid storing titles/descriptions directly in event payloads when entity IDs and reason codes suffice
- use tombstoned references or anonymized aggregate IDs only where audit law/policy requires retention
- crisis/security records require a dedicated deletion policy review

Event immutability does not mean personal data can never be deleted or anonymized.

---

## 27. Access Boundaries

Minimum access classes:

```txt
PRODUCT_RUNTIME
USER_AUDIT_VIEW
OPERATIONS_SUPPORT
SECURITY_RESTRICTED
ANALYTICS_AGGREGATED
```

Rules:

- users may access their own meaningful product history where exposed by product design
- ordinary support should not automatically access raw prompts or crisis content
- analytics uses minimized structured fields
- security/crisis records require elevated access and access logging
- cross-user joins must not expose identifiable narratives
- provider/model debugging access is time-bounded and auditable

---

## 28. Derived Analytics Boundary

Safe derived summaries include:

- counts of completed/dropped Tasks
- Routine occurrence completion rate over an explicit window
- carry count derived from placement-change events
- Reconcile session completion rate
- proposal acceptance/edit/rejection rates
- stale confirmation rate
- fallback usage rate
- invalid-output rate
- mutation-without-confirmation rate, expected zero

Unsafe or misleading persisted truth includes:

- Goal progress percentage inferred from Task completion
- Goal achievement probability
- psychological interpretation of missed work
- hidden productivity score used as recommendation authority
- AI confidence presented as deterministic evidence

Derived summaries must retain:

- calculation definition/version
- observation window
- included entity scope
- source event/fact version

A derived metric may be recomputed; it must not overwrite canonical entity meaning.

---

## 29. Metrics and Guardrail Measures

### Planning

- draft validation success rate
- draft approval rate
- edit-before-approval rate
- explicit rejection rate
- proposal expiry-without-decision rate
- partial-output rejection rate
- stale-draft invalidation rate

### Reconcile

- sessions opened/completed/abandoned
- rule-match frequency by rule version
- recommendation acceptance/edit/rejection/expiry rate
- sessions completed without AI explanation
- deterministic quick-action completion rate

### Safety and trust

- mutation without current confirmation, expected zero
- unsupported mutation rate, expected zero
- free-text Reconcile context rate, expected zero
- crisis PlanningDraft generation rate, expected zero
- unauthorized action rate, expected zero
- silent flow-affecting failure rate, expected zero
- hostile-content blocked rate
- secret-redaction incident rate

### Reliability

- stale command conflict rate
- idempotent replay rate
- no-op success rate
- cascade rollback rate
- outbox delivery retry rate
- duplicate event publication rate, expected zero semantically

---

## 30. Minimum Index and Uniqueness Requirements

Conceptual requirements:

```txt
UNIQUE(eventId)
UNIQUE(userId, planningDraftId, revisionNumber)
UNIQUE(userId, idempotencyKey) — from 019B
UNIQUE(proposalId, clientStableKey)
UNIQUE(ruleMatchId, entityType, entityId, role)
UNIQUE(outbox.eventId)
```

Useful indexes:

- events by `(userId, occurredAt)`
- events by `(aggregateType, aggregateId, occurredAt)`
- events by `correlationId`
- events by `transactionId`
- AI operations by `(userId, startedAt)`
- AI operations by `(feature, status, startedAt)`
- proposals by `(userId, feature, createdAt)`
- Reconcile sessions by `(userId, status, openedAt)`
- rule matches by `(sessionId, ruleId, ruleVersion)`
- outbox unpublished rows by `(publishedAt, nextAttemptAt)`

Indexes must not encourage broad raw-content querying.

---

## 31. Scenario Checks

### Scenario A — AI prepares a plan, user edits two items, then approves

Expected:

- AIOperation records generation
- PlanningDraft revision 1 stores validated structure
- revision 2 stores user-edited structure
- proposal items record explicit accepted-edited outcomes
- canonical creation events link to confirmation and proposal
- AI remains provenance, not actor

### Scenario B — User closes draft without deciding

Expected:

- draft eventually expires
- proposal outcome becomes expired-without-decision
- items are not mislabeled rejected

### Scenario C — Reconcile AI explanation fails

Expected:

- ReconcileSession remains valid
- deterministic facts and rule matches remain
- AI operation records failure/degraded mode
- no AI explanation content is required
- user may complete deterministic actions

### Scenario D — User accepts recommendation but command conflicts

Expected:

- recommendation decision records acceptance attempt
- command records conflict
- no mutation event exists
- recommendation is not reported as successfully applied
- refreshed proposal may supersede prior proposal

### Scenario E — Project completion stops three Routines

Expected:

- one PROJECT_COMPLETED event
- three ROUTINE_STOPPED events
- one CASCADE_COMPLETED event
- same transaction and correlation IDs
- outbox intent committed atomically

### Scenario F — Prompt injection appears in imported text

Expected:

- blocked event stores category/source, not full text
- no permission change
- restricted raw reference only if debug policy permits

### Scenario G — Crisis content is detected

Expected:

- no PlanningDraft
- safety-flow event created under restricted access
- no durable psychological profile
- resource locale may be recorded minimally

### Scenario H — User deletes account

Expected:

- raw prompts and draft content deleted/anonymized according to policy
- restricted records follow dedicated deletion rules
- ordinary analytics retain no identifiable narrative

### Scenario I — Outbox publishes duplicate delivery

Expected:

- same eventId
- consumer handles idempotently
- no duplicate semantic domain event

---

## 32. Stabilized Proposals Pending Claude Review

- canonical state remains authoritative for current truth
- domain/audit events preserve accepted historical facts
- AI operational records are separate from domain events
- raw AI content is optional, restricted, and not needed for ordinary analytics
- PlanningDraft is temporarily persisted with versioned revisions
- ReconcileSession is persisted explicitly
- absence is not treated as rejection
- AI is proposal provenance, never authorizing actor
- provider/model/prompt metadata remains outside canonical domain rows
- cascades produce parent and child events linked by transaction/correlation IDs
- transactional outbox is required
- crisis and hostile-content observability minimizes narrative content
- analytics may compute execution facts but may not persist unsupported Goal progress or psychological inference

---

## 33. Open Questions for Claude

### Q1. Are the four persistence layers separated correctly?

Review whether domain/audit and AI operational records are still accidentally duplicating authority.

### Q2. Should PlanningDraft revisions be persisted as full structured snapshots or deltas?

Current proposal: full bounded structured snapshots for MVP simplicity.

### Q3. Is explicit ReconcileSession persistence necessary?

Current proposal: yes, because event-only derivation creates ambiguous session boundaries.

### Q4. Is `IGNORED` correctly rejected as an item-level decision?

Current proposal: absence becomes proposal expiry-without-decision, not rejection.

### Q5. Should explicit acceptance be recorded before command outcome?

Review how to represent “user accepted, but mutation conflicted” without falsely reporting success.

### Q6. Is the event envelope minimal but sufficient?

Focus on event version, payload version, transaction, correlation, causation, proposal, confirmation, and rule linkage.

### Q7. Should lifecycle cascades emit both parent and child events?

Current proposal: yes, plus one operational cascade-completed event.

### Q8. Is RoutineOccurrence exposure best represented by immutable events, a `firstExposedAt` projection, or both?

Current proposal: event plus optional projection for efficient transaction checks.

### Q9. Are provider/model/prompt profile abstractions too heavy for MVP?

Review whether stable IDs and versions are justified without coupling domain tables to vendors.

### Q10. Are retention classes coherent without exact durations?

Review whether any class needs a different deletion or access policy.

### Q11. Is raw prompt/response retention too permissive or too restrictive?

Current proposal: disabled or short-lived by default, separately controlled, never required for ordinary analytics.

### Q12. Are crisis and hostile-content events minimized enough?

Focus on avoiding durable sensitive narratives and hidden user profiles.

### Q13. Are any proposed events actually UI telemetry rather than durable product/audit facts?

Remove event types that do not serve audit, correctness, safety, or validated analytics.

### Q14. Should `COMMAND_RECEIVED` and `TRANSACTION_ROLLED_BACK` live only in operational logs rather than the durable domain event store?

Current proposal: yes, unless needed for specific audit flows.

### Q15. Are accepted/edited/rejected/cancelled/expired outcomes sufficient?

Review Planning and Reconcile semantics, especially conflicts after acceptance.

### Q16. Does privacy deletion remain coherent with immutable audit history?

Review anonymization, tombstoning, and removal of user-authored content.

### Q17. Are derived analytics definitions sufficiently versioned?

Review observation windows, source versions, and prevention of misleading Goal progress.

### Q18. What minimum indexes are correctness-critical versus merely performance-oriented?

Focus on uniqueness, proposal item identity, event correlation, and outbox publication.

---

## 34. Mind Map Impact — Proposed, Do Not Apply Yet

### Product Vision

- AI-assisted decisions are explainable without storing unnecessary private content.
- current truth, historical fact, AI diagnostics, and raw content remain distinct.
- absence is not misrepresented as rejection.
- analytics describe execution without inventing Goal outcome or psychology.

### MVP Core Loop

```txt
scoped AI operation
→ validated proposal or deterministic facts
→ presented preview
→ explicit user decision
→ command and transaction
→ canonical mutation
→ durable audit/domain events
→ minimized analytics
```

### User Flow

Add:

- recoverable versioned PlanningDraft review
- expired-without-decision state
- accepted-but-conflicted state
- ReconcileSession continuity across several actions
- user-visible meaningful action history
- privacy deletion of draft/raw content

### AI Responsibilities

- emit schema-valid structured proposals
- preserve proposal/item identity
- expose model/prompt version through operational metadata
- avoid free-text Reconcile facts
- never claim command success from recommendation acceptance

### AI Guardrails

- no AI actor authority
- no raw sensitive content in ordinary events
- no absence-as-rejection inference
- no unsupported Goal progress metric
- no psychological scoring
- no crisis narrative in ordinary analytics
- no provider metadata coupled to canonical entities

### Data Events

Add the accepted taxonomy from Sections 19–22 after review.

### Traction and Guardrail Metrics

Add the metrics from Section 29 with explicit definition/version/window metadata.

### Current Decisions

Proposed:

- persisted PlanningDraft revisions
- explicit ReconcileSession
- provider-neutral AIOperation records
- transactional outbox
- parent and child cascade events
- retention classes by sensitivity and purpose

### Open Questions

- full snapshot versus delta draft revisions
- exposure event versus projection
- raw-content debug retention
- exact event-store versus operational-log split
- privacy deletion/anonymization mechanics

---

## 35. Affected Formal Documents — Record Only

After closure, update or create:

- event-envelope specification
- domain/audit event taxonomy
- PlanningDraft revision persistence specification
- proposal and decision outcome specification
- ReconcileSession and rule-match persistence specification
- AI operational observability specification
- provider-neutral model/prompt version ADR
- transactional outbox event publication specification
- retention and deletion classification policy
- hostile-content and crisis observability access policy
- analytics definition/version contract
- validation plan in Discussion 021
- rollout and data-migration plan in Discussion 022

---

## 36. Structured Review Request for Claude

Review this exact proposal for audit correctness, privacy minimization, semantic honesty, analytics validity, and proportional MVP complexity.

For every finding, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected section or record/event
4. concrete failure, privacy, or misleading-analytics scenario
5. why the proposal is unsafe, contradictory, excessive, or incomplete
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- event versus canonical truth boundaries
- AI actor/provenance separation
- PlanningDraft revision retention
- explicit versus inferred proposal outcomes
- accepted-but-conflicted representation
- ReconcileSession necessity
- event envelope and correlation/causation
- cascade event granularity
- exposure/invalidation observability
- provider-neutral versioning
- raw content retention
- crisis and hostile-content minimization
- privacy deletion versus immutable audit
- derived analytics honesty
- event types that should be operational-only

Do not expand into specific ORM APIs, dashboard styling, vendor selection, exact legal retention durations, warehouse technology, or final crisis-response wording.

This discussion remains open until Claude reviews this proposal and Mahdi resolves accepted findings.
