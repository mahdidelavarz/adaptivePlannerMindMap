# Discussion 020C — Structured Output, Reliability, and Cost Controls

## Status

Open for GPT × Claude review.

This document is Part C of Discussion 020.

Companion discussions:

- [[01-Open-Discussions/020-api-and-ai-runtime-architecture]]
- [[01-Open-Discussions/020a-final-ai-runtime-boundaries-and-orchestration-resolution]]
- [[01-Open-Discussions/020b-final-api-and-frontend-state-contracts-resolution]]

Accepted dependencies:

- [[01-Open-Discussions/018c-final-failure-privacy-domain-and-hostile-input-resolution]]
- [[01-Open-Discussions/019c-final-events-ai-observability-and-retention-resolution]]
- [[01-Open-Discussions/019c-amendment-ai-context-scope-observability]]
- [[01-Open-Discussions/020a-final-ai-runtime-boundaries-and-orchestration-resolution]]
- [[01-Open-Discussions/020b-final-api-and-frontend-state-contracts-resolution]]

---

## 1. Scope

This discussion defines how AI operations are validated, repaired, rejected, retried, timed out, rate-limited, versioned, observed, and cost-controlled without weakening accepted product, privacy, confirmation, or transaction guarantees.

It covers:

- structured-output validation pipeline
- deterministic repair allowlist
- semantic-policy validation
- retry classification and retry budgets
- timeout layers
- circuit breakers
- concurrency and rate limits
- provider fallback policy
- prompt, schema, catalog, model, and context-builder version resolution
- token and context budgets
- truncation and omission behavior
- per-operation cost ceilings
- operational observability
- kill switches
- user-facing failure mapping
- resilience test requirements

---

## 2. Explicitly Out of Scope

This discussion does not redefine:

- endpoint naming or client resource shapes from 020B
- canonical transactions or idempotent command execution from 019B
- AI port boundaries from 020A
- product behavior from Discussions 012–018
- final production threshold values
- cloud vendor, deployment topology, or secret-management provider
- legal retention durations
- multi-provider automatic fallback for the pilot

Thresholds in this document are policy shapes and configuration requirements. Exact numeric values are finalized during implementation and rollout planning in Discussions 021 and 022.

---

## 3. Governing Reliability Principle

An AI result becomes product-usable only after every required deterministic gate succeeds.

```txt
provider response
→ transport completion
→ decode
→ structural parse
→ schema validation
→ deterministic normalization/repair
→ schema validation again
→ domain vocabulary validation
→ product-policy validation
→ reference and ownership validation
→ consequence completeness validation
→ persist operational result
→ create reviewable proposal or bounded explanation
```

Failure at any required gate means:

```txt
no reviewable partial result
no confirmation resource
no canonical mutation
```

Retries may attempt to produce a new candidate output. Retries never relax validation rules.

---

## 4. Output Contract Families

The runtime recognizes three distinct output families.

### 4.1 Planning Proposal Output

Planning output is a complete structured proposal matching a versioned schema.

It may contain:

- proposed Goal, Project, Task, and Routine structures allowed by accepted product semantics
- explicit ownership references
- user-visible rationale fields where permitted
- consequence-preview inputs
- source references to user-provided constraints

It may not contain:

- executable commands
- repository instructions
- tool calls
- hidden mutation directives
- free-form reviewDate invention
- partial entity sets presented as complete

### 4.2 Reconcile Explanation Output

Reconcile explanation output is bounded explanatory content generated only from deterministic facts, rule matches, and group records.

It may not:

- create new facts
- change rule outcomes
- infer raw user narrative
- redefine recommendation eligibility
- replace a failed deterministic Reconcile computation

### 4.3 Safety Classification Output

Safety classification uses a closed vocabulary defined by 020A.

```txt
classification
reasonCodes[]
policyVersion
confidenceBand?
```

Unknown classifications, unknown reason codes, or free-form labels are invalid.

---

## 5. Validation Pipeline

### 5.1 Gate 1 — Transport Completion

The provider call must finish according to the configured operation contract.

Invalid conditions include:

- connection failure
- incomplete body
- provider stream termination before the final structured payload
- timeout
- cancellation
- response exceeding configured byte limits

No downstream parsing occurs on an incomplete response.

### 5.2 Gate 2 — Content-Type and Envelope Validation

The adapter verifies:

- expected provider response envelope
- expected content type or structured-output channel
- one final candidate payload
- no unexpected tool/function-call object
- no multiple competing final outputs

Any tool/function-call object is a final architecture violation, not a repairable model formatting issue.

### 5.3 Gate 3 — Structural Parse

The runtime parses the candidate into the declared output format.

For JSON-based output:

- the full payload must parse
- trailing commentary is rejected unless removed by an allowlisted purely syntactic repair
- multiple top-level objects are rejected
- duplicate keys are rejected
- excessive nesting or collection size is rejected

### 5.4 Gate 4 — Versioned Schema Validation

The parsed object is validated against the exact resolved schema version.

Validation includes:

- required fields
- allowed enum values
- field types
- minimum and maximum collection sizes
- string length limits
- date and timezone formats
- mutually exclusive fields
- forbidden additional properties where declared

The runtime never validates against “latest” after invocation. The operation is bound to the schema version resolved before provider invocation.

### 5.5 Gate 5 — Deterministic Normalization and Repair

Only the allowlisted repairs in Section 6 may run.

After repair, the entire payload is schema-validated again.

A repair operation must produce an auditable repair record containing:

```txt
repairCode
repairVersion
affectedPaths[]
semanticChangeExpected: false
```

### 5.6 Gate 6 — Domain Vocabulary Validation

The runtime verifies that values map to accepted domain concepts.

Examples:

- valid entity types
- accepted lifecycle statuses
- allowed ownership shapes
- accepted temporal source values
- valid recommendation and reason codes
- known warning codes
- valid safety classifications

Provider-created unknown domain codes are rejected, not dynamically registered.

### 5.7 Gate 7 — Product-Policy Validation

Policy validation enforces accepted decisions, including:

- Task and Routine ownership XOR rules
- no illegal child under terminal parent
- one Routine occurrence per Routine per local day in MVP
- reviewDate is user literal or deterministic default only
- Reconcile explanation cannot create facts
- protected or deadline-sensitive changes require accepted preview rules
- partial proposal subsets are not reviewable
- imported hostile content is not treated as instruction

### 5.8 Gate 8 — Reference and Scope Validation

Every referenced canonical ID must:

- exist where existence is required
- belong to the authenticated user/workspace
- be within the context scope manifest
- match the expected entity type
- not grant access to an entity omitted from the approved context

The model cannot expand its authority by returning an arbitrary ID.

### 5.9 Gate 9 — Completeness and Consequence Validation

Before a proposal becomes REVIEWABLE, the application verifies:

- all required proposal sections are present
- referenced consequences can be deterministically computed
- selected entity IDs are explicit
- expected versions can be bound later through confirmation
- no hidden unsupported action remains
- no warning acknowledgement is fabricated

### 5.10 Gate 10 — Persistence Boundary

A PlanningDraft revision is persisted only after all required gates succeed.

Failed candidate outputs may produce operational metadata, but never a partially reviewable draft.

---

## 6. Deterministic Repair Allowlist

Repair exists only to correct representation defects that do not alter meaning.

### 6.1 Allowed Repairs

Proposed MVP allowlist:

```txt
REMOVE_MARKDOWN_CODE_FENCE
REMOVE_LEADING_OR_TRAILING_NON_PAYLOAD_WHITESPACE
NORMALIZE_UNICODE_WHITESPACE
NORMALIZE_KNOWN_ENUM_CASE
NORMALIZE_ISO_DATE_FORMAT_WHEN_UNAMBIGUOUS
REMOVE_TRAILING_JSON_COMMA
```

Conditions:

- the repair must be deterministic
- the repair must not choose between multiple meanings
- the repaired value must still pass full validation
- the original candidate must never be shown to the user
- all repairs are recorded in AIOperation metadata

### 6.2 Forbidden Repairs

The runtime must not:

- invent missing required fields
- infer missing dates
- choose a parent entity
- rewrite user intent
- drop invalid proposal items and keep the rest
- coerce unknown enum values to the nearest known value
- merge duplicate entities semantically
- replace unknown IDs
- turn invalid recommendations into generic advice
- ask a second model to “fix” content without a new full operation attempt

### 6.3 Repair Result

```txt
repair succeeds + all gates pass
→ candidate may continue

repair fails or ambiguity exists
→ candidate rejected
```

---

## 7. Failure Classification

Every failure maps to one stable internal class.

```txt
CLIENT_INPUT_INVALID
CONTEXT_BUILD_FAILED
SAFETY_ROUTING_BLOCKED
PROVIDER_RATE_LIMITED
PROVIDER_AUTH_FAILED
PROVIDER_UNAVAILABLE
PROVIDER_TIMEOUT
PROVIDER_PROTOCOL_ERROR
OUTPUT_PARSE_FAILED
OUTPUT_SCHEMA_INVALID
OUTPUT_POLICY_INVALID
OUTPUT_REFERENCE_INVALID
OUTPUT_INCOMPLETE
OUTPUT_TOO_LARGE
CANCELLED_BY_USER
CANCELLED_BY_SYSTEM
CIRCUIT_OPEN
SYSTEM_RATE_LIMITED
BUDGET_EXCEEDED
KILL_SWITCH_DISABLED
INTERNAL_VALIDATION_ERROR
```

Failure classes are operational facts. They do not expose provider-specific error text to clients.

---

## 8. Retry Policy

### 8.1 General Rules

- retry applies only to the AI generation or explanation attempt
- retry does not create or authorize a mutation
- retry reuses the same logical PlanningAttempt or Reconcile explanation request
- each provider invocation receives a distinct attempt number and operation record
- cancellation stops further retry scheduling
- the retry budget is bounded per logical operation
- all validation gates rerun for every candidate

### 8.2 Retryable Failures

Potentially retryable, subject to budget and current circuit state:

```txt
PROVIDER_RATE_LIMITED
PROVIDER_UNAVAILABLE
PROVIDER_TIMEOUT
PROVIDER_PROTOCOL_ERROR
OUTPUT_PARSE_FAILED
OUTPUT_SCHEMA_INVALID
```

`OUTPUT_PARSE_FAILED` and `OUTPUT_SCHEMA_INVALID` permit at most one corrective retry in the pilot.

The corrective retry may include concise machine-generated validation feedback, such as:

```txt
schema path
expected type
received type
missing required field names
```

It must not include a request to reinterpret user intent.

### 8.3 Non-Retryable Failures

```txt
CLIENT_INPUT_INVALID
CONTEXT_BUILD_FAILED
SAFETY_ROUTING_BLOCKED
PROVIDER_AUTH_FAILED
OUTPUT_POLICY_INVALID
OUTPUT_REFERENCE_INVALID
OUTPUT_INCOMPLETE caused by semantic omission
OUTPUT_TOO_LARGE after budget enforcement
CANCELLED_BY_USER
KILL_SWITCH_DISABLED
BUDGET_EXCEEDED
```

`INTERNAL_VALIDATION_ERROR` is not retried automatically because it may indicate a product defect.

### 8.4 Retry Count Policy

Pilot proposal:

```txt
maximum provider invocations per logical operation: 2
```

Meaning:

- initial invocation
- at most one retry

No nested SDK retry may operate invisibly beyond this application-level budget.

Provider SDK automatic retry must be disabled or configured so the total physical request count remains observable and within the same budget.

### 8.5 Backoff

Retry uses bounded exponential backoff with jitter for transient provider failures.

Provider-supplied retry-after values may be respected only within the operation’s total deadline.

A retry is skipped when insufficient total-deadline budget remains.

---

## 9. Timeout Model

Timeouts are layered and configured per operation class.

```txt
connectionTimeout
responseStartTimeout
providerInvocationTimeout
logicalOperationDeadline
clientPollingExpiry
```

### 9.1 Connection Timeout

Limits establishment of the provider connection.

### 9.2 Response-Start Timeout

Limits how long the runtime waits for the provider to begin producing a response or structured result.

This is operational only. Partial provider output remains unusable.

### 9.3 Provider Invocation Timeout

Limits one physical provider call.

When exceeded:

- the adapter attempts provider cancellation where supported
- the invocation is marked timed out
- late results are discarded under 020A

### 9.4 Logical Operation Deadline

Covers all attempts, backoff, parsing, repair, and validation for one PlanningAttempt or explanation request.

No retry may begin after this deadline.

### 9.5 Polling Expiry

The API resource may remain readable after the operation finishes or fails, but the client stops active polling according to 020B terminal states.

Polling expiry is not the same as provider timeout.

---

## 10. Circuit Breaker Policy

Circuit breakers operate by:

```txt
provider configuration key
operation family
```

Planning and Reconcile explanation may use separate circuits because their failure rates and degraded behavior differ.

### 10.1 Circuit Inputs

Count toward opening:

- provider unavailability
- provider timeout
- provider protocol failures
- sustained provider rate limiting

Do not count:

- client validation failures
- policy rejection
- user cancellation
- domain-reference rejection
- expected schema rejection caused by a bad prompt version, which should trigger a version kill switch instead

### 10.2 Open-Circuit Behavior

Planning:

```txt
no provider invocation
→ PlanningAttempt terminal failure/degraded state
→ preserve user input
→ manual planning remains available
```

Reconcile explanation:

```txt
deterministic facts and rule matches remain available
→ explanation absent
→ facts-only degraded mode
```

Safety classification:

```txt
fail closed for AI generation eligibility
→ route to deterministic safe handling
```

### 10.3 Half-Open Probes

Only controlled system probes or a bounded subset of live requests may test recovery.

A user retry must not force the circuit closed.

### 10.4 Breaker State Observability

State changes are operational events:

```txt
CIRCUIT_OPENED
CIRCUIT_HALF_OPENED
CIRCUIT_CLOSED
```

They must not include raw prompt content.

---

## 11. Rate-Limit and Concurrency Policy

The system applies independent limits at multiple layers.

### 11.1 Per-User Limits

- Planning attempt creation rate
- concurrent active PlanningAttempt count
- Reconcile explanation request rate
- safety-classification request rate

### 11.2 Per-Workspace Limits

Where workspaces exist, aggregate limits prevent one workspace from exhausting system capacity.

### 11.3 System and Provider Limits

- global concurrent provider calls
- concurrent calls by operation family
- provider configuration quota
- queue or executor capacity

### 11.4 Duplicate Suppression

`clientAttemptId` idempotency from 020B prevents duplicate Planning invocation.

Equivalent in-flight requests with different identities are not automatically merged in MVP because their user intent and lifecycle identities differ.

### 11.5 Limit Result

Rate-limited requests receive a stable failure/resource state with:

```txt
failureClass
retryAllowed
retryAfterSeconds?
```

The system never claims the provider was called when the request was rejected locally.

---

## 12. Provider Fallback Policy

Automatic cross-provider fallback is disabled for the pilot.

Automatic fallback to another model under the same provider is also disabled.

Reasons:

- model behavior changes
- schema adherence differs
- safety characteristics differ
- cost predictability weakens
- comparison and audit become harder
- retry plus fallback can multiply physical calls

A provider or model change occurs only through an explicit runtime configuration rollout.

An existing logical operation does not silently switch provider/model after failure.

---

## 13. Runtime Artifact Version Registry

Every AI operation resolves immutable versions before invocation.

Required logical artifacts:

```txt
operationPolicyKey + operationPolicyVersion
promptTemplateKey + promptTemplateVersion
outputSchemaKey + outputSchemaVersion
contextBuilderKey + contextBuilderVersion
ruleCatalogVersion where applicable
safetyPolicyVersion
providerConfigurationKey + providerConfigurationVersion
modelIdentifier
repairPolicyVersion
```

### 13.1 Resolution Rule

```txt
logical operation starts
→ resolve exact artifact versions
→ persist version references on AIOperation
→ invoke provider
→ never switch versions inside the same logical operation
```

A retry normally uses the same resolved versions.

A corrective schema retry may use a retry-variant prompt template only if that variant and version were resolved as part of the original operation policy.

### 13.2 Immutable Registry Entries

Published versions are immutable.

Changes create new versions rather than modifying historical definitions.

### 13.3 Activation and Rollback

Logical configuration keys point to active versions.

Activation changes affect only new logical operations.

Rollback selects a prior known version for new operations; it does not alter historical records.

---

## 14. Token and Context Budgets

Budgets are defined per operation family.

```txt
maximum input tokens
maximum output tokens
maximum canonical entities
maximum proposal items
maximum explanation groups
maximum raw input characters
maximum serialized context bytes
```

### 14.1 Planning Budget

Planning context is assembled by explicit priority, not arbitrary string truncation.

Proposed priority order:

```txt
1. user’s current request and explicit constraints
2. timezone and current local-date context
3. directly referenced entities
4. structural parents and required children
5. conflicting deadlines and protection facts
6. bounded relevant active entities
7. optional lower-priority summaries
```

### 14.2 Reconcile Budget

Reconcile sends only deterministic structured facts, rule matches, and bounded groups.

If deterministic results exceed explanation budget:

- groups are selected by deterministic priority
- omitted group counts are reported
- facts remain available through the non-AI resource
- the model is never told that the selected subset is complete

### 14.3 Forbidden Truncation

The runtime must not:

- cut serialized JSON at a character boundary
- remove random entities
- omit required parent relationships
- remove a warning or deadline fact while keeping the affected action
- truncate user input without disclosure
- silently change local-date context

### 14.4 Budget Overflow

When required context cannot fit coherently:

```txt
operation rejected or narrowed through explicit user choice
```

The system does not generate from an incoherent partial context.

---

## 15. Cost Controls

### 15.1 Cost Estimate Before Invocation

The runtime estimates cost using:

- resolved model configuration
- estimated input tokens
- maximum output tokens
- remaining retry budget

The estimate need not be billing-exact, but it must be conservative enough for policy enforcement.

### 15.2 Per-Operation Ceiling

Each operation family has a configured maximum estimated and actual cost.

If estimated cost exceeds the ceiling before invocation:

```txt
BUDGET_EXCEEDED
→ no provider call
```

### 15.3 Per-User and System Budgets

Optional pilot controls:

- rolling daily user budget
- workspace budget
- global daily budget
- emergency provider spend cap

Budget exhaustion must not disable manual product operation.

### 15.4 Retry Cost

The maximum retry cost is included in the logical operation budget.

The runtime must not spend the initial budget and then discover that the retry budget was imaginary.

### 15.5 Actual Usage Reconciliation

When the provider returns usage data, record:

```txt
inputTokens
outputTokens
cachedInputTokens?
providerReportedCost?
normalizedEstimatedCost
```

Missing provider usage metadata does not make the operation invalid, but it is recorded as incomplete observability.

---

## 16. Operational Observability

AIOperation records the minimum metadata required by 019C and its amendment.

Required or proposed fields:

```txt
operationId
logicalAttemptId
operationFamily
attemptNumber
providerConfigurationKey
providerConfigurationVersion
modelIdentifier
promptTemplateKey
promptTemplateVersion
outputSchemaKey
outputSchemaVersion
contextBuilderKey
contextBuilderVersion
contextScopeManifestId
ruleCatalogVersion?
safetyPolicyVersion
repairPolicyVersion
startedAt
completedAt?
latencyMs?
inputTokenCount?
outputTokenCount?
estimatedCost?
providerReportedCost?
resultClass
failureClass?
repairCodes[]
retryScheduled
circuitStateAtStart
rateLimitSource?
rawContentRetentionReference?
```

Observability must not include:

- unrestricted raw prompt text
- unrestricted raw response text
- full canonical entity snapshots
- user crisis profiling
- provider secret material

---

## 17. Kill Switches

Kill switches exist at least for:

```txt
all AI operations
Planning generation
Reconcile explanation
Safety classification provider path
specific provider configuration
specific model identifier
specific prompt template version
specific output schema version
specific operation policy version
```

### 17.1 Behavior

Kill switch evaluation occurs before every physical provider invocation, including retries.

If disabled:

```txt
no provider call
→ stable KILL_SWITCH_DISABLED result
→ accepted degraded/manual behavior
```

### 17.2 Audit

Every switch change records:

- actor
- previous state
- new state
- scope
- reason code
- timestamp
- change ticket/reference where applicable

### 17.3 Testing

Automated tests verify:

- disabled switch prevents provider invocation
- in-flight operation does not schedule a retry after disablement
- Planning falls back to manual path
- Reconcile retains deterministic facts-only mode
- no canonical mutation depends on switch recovery

---

## 18. User-Facing Failure Mapping

Clients receive stable product-facing categories rather than provider internals.

```txt
INPUT_NEEDS_CORRECTION
AI_TEMPORARILY_UNAVAILABLE
AI_REQUEST_TIMED_OUT
AI_RESULT_COULD_NOT_BE_VALIDATED
AI_REQUEST_LIMIT_REACHED
AI_BUDGET_LIMIT_REACHED
AI_FEATURE_DISABLED
REQUEST_CANCELLED
RECONCILE_EXPLANATION_UNAVAILABLE
RECONCILE_UNAVAILABLE
```

Rules:

- preserve user-authored input where allowed by 020B
- explain whether retry is available
- avoid claiming that retry will succeed
- do not expose provider name unless product policy deliberately does so
- do not expose raw safety classification details
- always keep manual/deterministic paths visible where accepted

---

## 19. Failure Matrix

```txt
Failure                          Retry?   Planning Result                     Reconcile Result
------------------------------------------------------------------------------------------------------------
Client input invalid             No       input correction required          input correction required
Context build failed             No       unavailable, manual remains         unavailable
Provider unavailable             Once     temporary failure                   facts-only mode
Provider timeout                 Once     timeout                             facts-only mode
Provider rate limited            Once*    temporary/rate-limited              facts-only mode
Parse/schema invalid             Once     validation failure if retry fails   explanation absent
Policy/reference invalid         No       rejected candidate                  explanation absent
User cancellation                No       cancelled                           explanation cancelled
Circuit open                     No       AI unavailable                      facts-only mode
Budget exceeded                  No       budget-limited                      facts-only mode
Kill switch disabled             No       manual path                         facts-only mode
Deterministic Reconcile failure  No       not applicable                      Reconcile unavailable
```

`Once*` means only when retry-after and logical deadline permit it.

---

## 20. Scenario Checks

### Scenario A — Valid JSON with an Unknown Enum

Expected:

- schema/domain validation rejects it
- nearest enum is not guessed
- at most one corrective retry
- no draft if retry fails

### Scenario B — Provider Returns One Valid and One Invalid Proposal Item

Expected:

- entire candidate rejected
- valid item is not exposed as a partial draft
- no confirmation can be created

### Scenario C — Markdown Fence Around Otherwise Valid JSON

Expected:

- allowlisted fence removal
- full schema validation reruns
- repair metadata recorded

### Scenario D — Missing Required reviewDate

Expected:

- repair does not invent it
- corrective retry may request schema compliance
- deterministic product default is applied only by accepted application logic where allowed, not by repair pretending the model supplied it

### Scenario E — Provider Timeout Then Late Success

Expected:

- timed-out invocation remains failed
- late result discarded
- retry uses a new physical operation record if budget allows

### Scenario F — User Cancels During Backoff

Expected:

- scheduled retry removed
- no later provider call
- resource ends CANCELLED

### Scenario G — Circuit Opens During Planning

Expected:

- new requests do not invoke provider
- current user input preserved
- manual planning remains available

### Scenario H — Circuit Opens for Reconcile Explanation

Expected:

- deterministic Reconcile facts remain accessible
- explanation is absent
- no raw-data AI fallback

### Scenario I — Required Context Exceeds Budget

Expected:

- no arbitrary truncation
- request fails or asks user to narrow scope explicitly
- omitted context is not silently represented as complete

### Scenario J — Duplicate Client Attempt

Expected:

- 020B idempotency returns existing PlanningAttempt
- no additional cost or provider invocation

### Scenario K — SDK Performs Hidden Retries

Expected:

- integration test detects physical call count above policy
- SDK retries disabled or brought under observable application budget

### Scenario L — Kill Switch Enabled Between Attempts

Expected:

- next retry does not invoke provider
- operation terminates with feature-disabled/degraded behavior

---

## 21. Proposed Automated Tests

Minimum categories:

- schema contract tests for every output version
- property tests for repair idempotence and semantic neutrality
- golden tests for prompt/schema compatibility
- invalid enum and unknown-property tests
- cross-user ID reference rejection tests
- partial-output rejection tests
- retry-budget tests
- hidden SDK retry detection
- timeout and late-result discard tests
- cancellation during invocation and backoff tests
- circuit-breaker transition tests
- rate-limit isolation tests
- token-budget prioritization tests
- coherent-context overflow tests
- cost-ceiling tests
- kill-switch enforcement tests
- observability redaction tests
- version rollback tests

---

## 22. Stabilized Proposals Pending Claude Review

- every AI result passes a deterministic multi-stage validation pipeline.
- repairs are syntactic, allowlisted, auditable, and semantically neutral.
- partial proposal salvage is forbidden.
- one retry maximum is proposed for the pilot.
- provider SDK retries must fit inside the same observable retry budget.
- automatic provider/model fallback is disabled.
- timeout layers are distinct and bounded by one logical deadline.
- circuit breakers degrade Planning to manual and Reconcile explanation to facts-only.
- context reduction is priority-based and coherent, never arbitrary truncation.
- exact runtime artifact versions are resolved before invocation and remain fixed for the logical operation.
- token and cost budgets are enforced before provider invocation where possible.
- kill switches are evaluated before every physical call.
- operational metadata is retained without ordinary raw-content persistence.

---

## 23. Open Questions for Claude

### Q1. Is the ten-gate validation pipeline correctly ordered?

Review whether repair should occur before initial schema validation or only after the first validation failure, and whether any correctness-critical gate is missing.

### Q2. Is the repair allowlist too permissive?

Pay special attention to trailing-comma removal, enum-case normalization, and unambiguous date normalization.

### Q3. Should parse/schema failure receive one corrective retry?

Review whether a retry with validation feedback is safe and worthwhile for the pilot.

### Q4. Are any failures classified as retryable that should be final?

Focus on provider protocol errors, rate limiting, output parse failure, and output schema invalidity.

### Q5. Is two physical provider invocations per logical operation the right pilot maximum?

Review cost, latency, and reliability tradeoffs.

### Q6. How should hidden SDK retries be mechanically prevented or measured?

Review configuration, HTTP instrumentation, and integration-test options.

### Q7. Are timeout layers coherent without first-token streaming in the product API?

Review whether `responseStartTimeout` is still useful for provider transport even though partial content is never exposed.

### Q8. Are circuit breakers correctly separated by provider configuration and operation family?

Review whether safety classification requires a separate circuit and fail-closed policy.

### Q9. Should output schema failures influence the provider circuit breaker?

Current proposal: no; they should influence prompt/schema-version health and kill switches instead.

### Q10. Are rate limits sufficiently separated from idempotency?

Review duplicate suppression, concurrent active attempt limits, and abusive distinct request identities.

### Q11. Is disabling all automatic fallback correct for the pilot?

Review whether any same-provider fallback is justified.

### Q12. Is retry-version pinning correct?

Current proposal: retries use the same versions, except a pre-resolved corrective prompt variant.

### Q13. Is the context-priority policy safe?

Review whether any required context category could be accidentally treated as optional.

### Q14. When required context does not fit, should the operation fail or split into multiple proposals?

Current proposal: fail or require explicit user narrowing; no automatic semantic splitting in MVP.

### Q15. Are cost controls proportional for the pilot?

Review pre-invocation estimates, rolling user budgets, and emergency system caps.

### Q16. Which observability fields risk becoming sensitive content by proxy?

Review manifests, reason codes, failure classes, token counts, and safety metadata.

### Q17. Are kill switches granular enough and mechanically enforceable?

Review prompt/schema/model/version switches and behavior for already-running operations.

### Q18. Does any failure path weaken accepted manual fallback, facts-only Reconcile, cancellation, privacy, or confirmation guarantees?

Review against Discussions 018C, 019C, 020A, and 020B.

---

## 24. Mind Map Impact — Proposed, Do Not Apply Yet

### Product Vision

- AI assistance is bounded by deterministic validation and visible fallback.
- provider instability never corrupts canonical user state.
- cost controls do not remove manual product value.

### MVP Core Loop

```txt
scoped context
→ version-pinned AI invocation
→ parse and schema validation
→ deterministic repair if eligible
→ policy/reference validation
→ complete reviewable result
→ explicit confirmation
```

### User Flow

Add:

- temporary AI unavailable state
- timeout state
- request limit state
- validation-failed state
- preserved-input recovery
- facts-only Reconcile explanation state
- explicit scope-narrowing path when coherent context exceeds budget

### AI Responsibilities

- return one complete result matching the declared schema
- remain inside known vocabulary
- never rely on repair to invent missing semantics
- explain only deterministic Reconcile facts

### AI Guardrails

- no partial salvage
- no hidden tool call
- no semantic repair
- no arbitrary context truncation
- no silent provider/model fallback
- no unbounded retry
- no invocation after kill switch or budget rejection

### Data Events / Operational Records

Candidates:

```txt
AI_VALIDATION_FAILED
AI_REPAIR_APPLIED
AI_RETRY_SCHEDULED
AI_RETRY_SKIPPED
AI_BUDGET_REJECTED
AI_RATE_LIMITED
AI_TIMEOUT
AI_CIRCUIT_OPENED
AI_CIRCUIT_CLOSED
AI_KILL_SWITCH_BLOCKED
AI_CONTEXT_TOO_LARGE
AI_USAGE_RECORDED
```

These remain operational unless 019C explicitly classifies a subset otherwise.

### Traction and Guardrail Metrics

- schema-valid-first-attempt rate
- repair rate by repair code
- retry success rate
- total physical calls per logical operation
- timeout rate
- circuit-open duration
- context-budget rejection rate
- estimated-versus-actual token variance
- cost per successful reviewable proposal
- duplicate invocation rate, expected near zero
- partial proposal exposure rate, expected zero
- provider fallback rate, expected zero
- tool-call output rate, expected zero

### Current Decisions

Proposed:

- one retry maximum
- no provider fallback
- deterministic repair allowlist
- version-pinned runtime artifacts
- coherent context budgets
- pre-invocation cost checks
- operation-family circuit breakers
- granular audited kill switches

### Open Questions

- exact pilot timeout values
- exact rate-limit thresholds
- exact token and cost ceilings
- whether one corrective schema retry is worth the latency
- which repair rules remain enabled after empirical testing

---

## 25. Affected Formal Documents — Record Only

After closure, update or create:

- structured-output schema contracts
- AI validation pipeline specification
- deterministic repair ADR
- retry and timeout policy
- circuit-breaker and rate-limit ADR
- runtime artifact registry specification
- token/context budget policy
- AI cost-control policy
- kill-switch runbook
- provider integration test plan
- rollout and alert thresholds in Discussion 022

---

## 26. Structured Review Request for Claude

Review this proposal for correctness, resilience, privacy, cost proportionality, and compatibility with accepted Discussions 018C, 019C, 020A, and 020B.

For every finding, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected section
4. concrete failure or abuse scenario
5. why the proposal is unsafe, contradictory, incomplete, or unnecessarily complex
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- validation gate ordering
- semantic neutrality of repairs
- retry amplification and hidden SDK retries
- schema retry safety
- timeout and late-response behavior
- circuit-breaker inputs
- rate-limit versus idempotency boundaries
- context-budget omission risks
- provider fallback policy
- version pinning
- cost estimation and budget enforcement
- observability privacy
- kill-switch enforcement
- degraded-mode consistency

Do not expand into endpoint naming, provider SDK code, final numeric thresholds, UI styling, cloud deployment topology, or canonical transaction semantics.

This discussion remains open until Claude reviews the proposal and Mahdi resolves accepted findings.
