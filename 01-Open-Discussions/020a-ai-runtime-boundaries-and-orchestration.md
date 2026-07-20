# Discussion 020A — AI Runtime Boundaries and Orchestration

## Status

Open for GPT × Claude review.

This is Part A of the Discussion 020 runtime architecture program.

Companion discussions:

- [[01-Open-Discussions/020-api-and-ai-runtime-architecture]]
- [[01-Open-Discussions/020b-api-and-frontend-state-contracts]]
- [[01-Open-Discussions/020c-structured-output-reliability-and-cost-controls]]

Accepted dependencies:

- [[01-Open-Discussions/013-ai-planning-entry-and-conversation-flow]]
- [[01-Open-Discussions/014-ai-planning-draft-and-approval-contract]]
- [[01-Open-Discussions/017b-final-reconcile-intelligence-resolution]]
- [[01-Open-Discussions/018a-final-action-permission-trust-and-reversibility-resolution]]
- [[01-Open-Discussions/018c-final-failure-privacy-domain-and-hostile-input-resolution]]
- [[01-Open-Discussions/019a-final-canonical-data-model-resolution]]
- [[01-Open-Discussions/019b-final-transactions-concurrency-and-idempotency-resolution]]
- [[01-Open-Discussions/019c-final-events-ai-observability-and-retention-resolution]]

---

## 1. Scope

This discussion defines the runtime boundaries and orchestration flow for Planning AI and Reconcile AI.

It decides:

- where AI integration lives in the application architecture
- how Spring AI is contained behind internal ports
- how provider-specific concepts are isolated
- how Planning and Reconcile requests are orchestrated
- how context is constructed and minimized
- whether output delivery is synchronous or streamed
- how cancellation behaves
- how kill switches and degraded modes route requests
- whether provider fallback exists in the pilot
- how AI output remains separated from canonical mutation

This discussion does not define endpoint names, final DTOs, retry counts, timeout thresholds, cost ceilings, or implementation milestones.

---

## 2. Governing Runtime Principle

```txt
AI runtime
→ proposes or explains
→ never owns canonical mutation
```

The runtime may:

- construct scoped context
- invoke an approved model configuration
- receive structured output
- return draft or explanation candidates
- record provider-neutral operational metadata

The runtime may not:

- write canonical Goal, Project, Task, Routine, or RoutineOccurrence rows
- execute lifecycle transitions
- bypass confirmation
- decide authorization
- decide transaction success
- treat model output as current canonical truth
- expose repository, database, or unrestricted tool access to the model

Canonical mutations remain deterministic application commands governed by Discussions 018A and 019B.

---

## 3. Proposed Architectural Shape

```txt
Client/API Adapter
→ Application Use Case
→ AI Orchestrator
   ├── Context Builder
   ├── Policy Gate
   ├── Prompt/Schema Registry Port
   ├── AI Capability Port
   ├── Output Validation Port
   └── AI Observability Port
→ Spring AI Adapter
→ Provider Adapter / Provider API
```

Canonical mutation is a separate path:

```txt
confirmed proposal/action
→ deterministic command use case
→ revalidate current state
→ transaction/idempotency layer
→ canonical mutation + outbox
```

The AI orchestration path and command mutation path may correlate through IDs, but they are not one authority boundary.

---

## 4. Layer Responsibilities

### 4.1 Domain Layer

The domain layer contains:

- canonical entities and lifecycle rules
- permission and consequence semantics
- deterministic invariants
- value objects and domain decisions

The domain layer must not import or reference:

- Spring AI types
- provider SDK types
- prompt templates
- token usage concepts
- model names
- streaming chunks
- provider error classes

### 4.2 Application Layer

The application layer owns:

- Planning and Reconcile use cases
- context scope decisions
- policy and authorization checks
- orchestration order
- correlation IDs
- draft/session persistence decisions
- selection of an internal AI capability
- mapping validated output into application proposal records
- routing to manual or deterministic fallback

### 4.3 AI Integration Layer

The AI integration layer owns:

- Spring AI integration
- provider request construction
- provider credentials
- provider-specific schema/structured-output configuration
- stream adaptation where used
- provider error normalization
- provider usage metadata extraction

### 4.4 Infrastructure Layer

Infrastructure owns:

- persistence adapters
- provider HTTP transport
- secrets/configuration
- observability sinks
- circuit breakers and rate limit implementation
- kill-switch storage and runtime configuration

---

## 5. Internal AI Ports

The application depends on narrow internal ports rather than Spring AI directly.

Proposed capability ports:

```txt
PlanningGenerationPort
ReconcileExplanationPort
```

Conceptual contracts:

```txt
PlanningGenerationPort.generate(PlanningGenerationRequest)
→ PlanningGenerationResult

ReconcileExplanationPort.explain(ReconcileExplanationRequest)
→ ReconcileExplanationResult
```

The request and result types are product-owned application contracts.

They must not expose:

- provider message classes
- provider tool-call objects
- Spring AI prompt types
- provider finish-reason enums
- raw provider exceptions

A shared lower-level provider gateway may exist inside the integration layer, but application code interacts with capability-specific ports.

### 5.1 Why Two Capability Ports

Planning and Reconcile have materially different boundaries:

```txt
Planning
→ may interpret scoped user request
→ returns complete structured proposal candidate

Reconcile
→ receives structured facts only
→ returns explanation/recommendation wording only
```

A single generic `ChatModelPort` would make it easier to accidentally send Planning context into Reconcile or use free-form chat behavior where a strict capability contract is required.

---

## 6. Spring AI Boundary

Proposed decision:

```txt
Application layer
→ internal capability ports
→ Spring AI adapters
→ provider
```

Spring AI is used as an integration framework, not as an application or domain abstraction.

Allowed Spring AI responsibilities:

- model client construction
- structured-output integration
- provider-neutral request execution where practical
- provider usage extraction
- provider switching inside adapters

Forbidden leakage:

- Spring AI annotations or response types in domain entities
- Spring AI client calls from controllers or domain services
- provider model names embedded in product rules
- provider errors returned directly to clients
- prompts stored as arbitrary strings beside domain logic

The adapter may expose normalized internal metadata such as:

```txt
providerKey
modelConfigKey
latencyMs
inputTokenCount?
outputTokenCount?
finishCategory
normalizedFailureCategory?
```

---

## 7. Provider-Neutral Configuration

The application selects a logical model configuration, not a literal provider model name.

Example:

```txt
PLANNING_MVP_PRIMARY
RECONCILE_TEXT_MVP_PRIMARY
```

A runtime registry resolves that logical key to:

```txt
provider
providerModel
temperature/configuration
schema version
prompt template version
maximum output budget
capability flags
```

Domain and application rules reference only logical capability/configuration keys where needed.

Provider-specific fields remain in adapter/configuration records and operational observability.

---

## 8. Planning AI Orchestration

Proposed flow:

```txt
1. authenticate user
2. authorize Planning flow
3. classify domain and safety boundary
4. build minimum scoped Planning context
5. create PlanningDraft attempt identity
6. resolve prompt/schema/model configuration versions
7. invoke PlanningGenerationPort
8. parse and validate complete output
9. apply only allowlisted deterministic normalization
10. reject partial, stale, unsafe, or semantically invented output
11. persist temporary draft revision and proposal metadata
12. return reviewable draft
```

No canonical entity is created during steps 1–12.

After explicit confirmation:

```txt
confirmed draft revision
→ independent deterministic command flow
→ reload current canonical state
→ revalidate permissions, versions, consequences, dates, ownership
→ commit or conflict
```

The AI invocation is not repeated automatically during commit.

### 8.1 Planning Context Builder

Allowed context follows Discussion 018C:

- current user request
- user-approved constraints
- timezone and current local date
- current unapproved draft revision where relevant
- scoped relevant entity summaries
- explicit availability or scheduling constraints
- known conflicting deadlines

Default exclusions:

- entire account history
- unrelated completed or dropped entities
- private communications
- hidden behavioral scoring
- sensitive profile fields unrelated to the request
- secrets or authentication material

The context builder must produce a typed internal object before prompt rendering.

Prompt templates must not independently query unrestricted repositories.

### 8.2 Context Scope Manifest

Each Planning AI operation should record a non-content manifest such as:

```txt
contextEntityTypes
contextEntityCount
includedFieldCategories
excludedSensitiveCategories
contextBuilderVersion
```

This supports observability without duplicating raw user content.

---

## 9. Reconcile AI Orchestration

Reconcile is deterministic-first.

Proposed flow:

```txt
1. authenticate user
2. compute deterministic facts
3. evaluate versioned rules
4. derive allowed actions and protection flags
5. build structured Reconcile groups
6. persist/identify ReconcileSession
7. if AI text is enabled, build structured-only explanation request
8. invoke ReconcileExplanationPort
9. validate explanation references and allowed action vocabulary
10. attach AI_EXPLANATION separately from FACT and RULE_MATCH
11. return deterministic actions regardless of AI text availability
```

### 9.1 Reconcile Context Boundary

For MVP, the AI request contains structured facts only:

- safe display labels where necessary
- entity type and scoped identifier
- matched rule IDs and versions
- reason codes
- observed metrics
- evidence quality
- protection flags
- deterministic consequences
- allowed actions

It excludes:

- notes
- descriptions
- imported messages
- emotional explanations
- free-text user narrative
- raw historical conversations

### 9.2 Reconcile AI Authority

Reconcile AI may:

- explain deterministic findings
- organize wording
- summarize allowed choices

It may not:

- add a new rule match
- remove a deterministic rule match
- alter allowed actions
- change severity
- infer diagnosis, motivation, or capacity
- generate a mutation command not present in the allowed action set

---

## 10. Model Tool Access

Proposed MVP decision:

```txt
No model-executed mutation tools.
No unrestricted read tools.
```

The model receives a prepared context payload and returns a candidate response.

If framework-level tool calling is used internally for a narrow read-only helper in the future, it requires a separate accepted decision and must preserve:

- explicit allowlist
- user/workspace scoping
- no secrets
- no mutation
- bounded result size
- hostile-content isolation
- complete auditability

No such tool access is required for the MVP proposal.

---

## 11. Synchronous Versus Streaming Behavior

### 11.1 Planning

Proposed MVP behavior:

```txt
request accepted
→ server performs bounded generation
→ client receives only final validated draft or failure
```

The provider may stream internally, but unvalidated partial model output is not exposed as an approvable draft.

Optional progress signals may be deterministic and content-free:

```txt
REQUEST_ACCEPTED
CONTEXT_PREPARED
GENERATING
VALIDATING
COMPLETED | FAILED | CANCELLED
```

These signals do not contain model text and do not imply success before validation finishes.

### 11.2 Reconcile

Deterministic FACT and RULE_MATCH data should be returned without waiting for AI explanation where practical.

Possible hybrid contract:

```txt
deterministic Reconcile result available
→ AI explanation pending
→ explanation attached when validated
```

If AI explanation fails, deterministic Reconcile remains usable.

### 11.3 Streaming Text

Direct token streaming to the user is not required for MVP.

Reasons:

- Planning partial output is unusable by policy
- streamed text complicates safety and schema validation
- cancellation semantics become harder to explain
- Reconcile deterministic output already provides immediate value

Streaming may be revisited later for non-authoritative explanation text.

---

## 12. Cancellation Semantics

Cancellation stops waiting for or continuing an AI generation attempt where technically possible.

It does not:

- roll back a canonical mutation, because AI generation performs none
- imply provider execution certainly stopped
- create an approvable partial draft
- silently discard the latest previously validated draft revision

Proposed state:

```txt
AI operation CANCEL_REQUESTED
→ adapter attempts cancellation
→ final operational state CANCELLED or COMPLETED_AFTER_CANCEL_REQUEST
```

If the provider completes after cancellation:

- the result is not automatically shown as current
- it is not persisted as a new reviewable revision unless the orchestration policy explicitly still accepts it
- it cannot trigger mutation
- operational metadata records the late completion

A user cancellation and a provider timeout are distinct outcomes.

---

## 13. Failure Routing

Normalized runtime outcomes:

```txt
SUCCESS
CANCELLED
PROVIDER_UNAVAILABLE
TIMEOUT
RATE_LIMITED
SAFETY_REFUSAL
MALFORMED_OUTPUT
SCHEMA_INVALID
PARTIAL_OUTPUT
STALE_CONTEXT
POLICY_REJECTED
INTERNAL_PROCESSING_ERROR
```

The adapter maps provider-specific errors into normalized categories.

The orchestrator then applies accepted product behavior:

```txt
AI failure
→ no canonical mutation
→ preserve user-authored input
→ preserve current canonical state
→ show user-facing status
→ offer retry or manual/deterministic fallback
```

Provider errors do not escape directly to the client.

---

## 14. Kill Switches and Degraded Modes

Required controls from Discussion 018C:

```txt
AI_GLOBAL_KILL_SWITCH
PLANNING_AI_KILL_SWITCH
RECONCILE_AI_TEXT_KILL_SWITCH
PROVIDER_SPECIFIC_KILL_SWITCH
READ_ONLY_DEGRADED_MODE
```

Routing rules:

### AI_GLOBAL_KILL_SWITCH enabled

- no Planning or Reconcile AI provider invocation
- manual Planning remains available
- deterministic Reconcile remains available
- canonical read/write product flows remain available where otherwise allowed

### PLANNING_AI_KILL_SWITCH enabled

- AI Planning entry returns unavailable state
- manual Goal/Project/Task/Routine creation remains available
- existing PlanningDrafts remain reviewable only if still valid under retention and staleness rules

### RECONCILE_AI_TEXT_KILL_SWITCH enabled

- FACT remains visible
- RULE_MATCH remains visible
- AI_EXPLANATION is absent
- deterministic actions remain available

### PROVIDER_SPECIFIC_KILL_SWITCH enabled

- affected provider adapter is unavailable
- fallback occurs only if an accepted fallback policy permits it
- no silent provider substitution that changes privacy or quality assumptions

Kill-switch checks occur before provider invocation and are recorded operationally.

---

## 15. Provider Fallback Policy

Proposed pilot decision:

```txt
No automatic cross-provider fallback by default.
```

Reasons:

- providers may differ in privacy, residency, schema behavior, and safety handling
- fallback can obscure incidents
- prompt/schema compatibility must be validated per provider
- pilot complexity should remain bounded

Permitted fallback forms:

1. retry the same logical configuration under the retry policy defined in 020C
2. use a pre-approved alternate configuration only when:
   - privacy and retention terms are accepted
   - schema compatibility is validated
   - prompt/version mapping is explicit
   - observability records the actual provider/model configuration
   - kill-switch policy permits it

The user-facing product need not expose provider brand names, but internal observability must record the actual route.

Claude should review whether a pre-approved same-provider secondary model is justified for the pilot.

---

## 16. Safety and Domain Classification Placement

Domain/safety classification occurs before generation.

```txt
request
→ operation-scoped classification
→ allowed Planning/Reconcile path or safety boundary
```

Classification output may select:

- normal planning flow
- high-risk organization-only boundary
- refusal/manual-safe boundary
- crisis safety flow

The model used for proposal generation is not the sole enforcement mechanism.

Safety enforcement must include deterministic application policy before and after model invocation.

Crisis handling remains governed by the explicit Discussion 021 release gate.

No crisis-triggered data may be routed into ordinary PlanningDraft generation.

---

## 17. Hostile and Imported Content Boundary

All user-authored and imported content is untrusted data.

The runtime must structurally separate:

```txt
system/application instructions
from
user/imported content
```

Imported content cannot:

- alter system policy
- grant tools or permissions
- request secrets
- redefine allowed actions
- override schema
- become runtime configuration

The context builder:

- scopes imported content to the requested operation
- labels it as untrusted content
- truncates or excludes unnecessary content
- never follows embedded instructions as authority
- avoids sending attachments or links unless explicitly supported and sanitized

---

## 18. Observability Integration

Every AI operation creates provider-neutral operational metadata under Discussion 019C.

Minimum linkage:

```txt
operationId
user/workspace scope
capability: PLANNING | RECONCILE_EXPLANATION
correlationId
planningDraftId? / reconcileSessionId?
promptTemplateVersion
schemaVersion
contextBuilderVersion
logicalModelConfigKey
actual provider/model route
startedAt
completedAt?
normalized outcome/failure category
latency and token metadata where available
```

The runtime does not retain raw prompt/response content by default merely because the provider SDK exposes it.

Crisis events remain SECURITY_RESTRICTED and are excluded from ordinary analytics aggregation.

---

## 19. Secrets and Configuration

Provider credentials:

- remain server-side
- are never included in prompts, logs, client responses, or domain records
- are referenced through secret/configuration infrastructure
- support rotation without changing domain behavior

Prompt templates, schema definitions, and logical model configuration are versioned deployable artifacts or controlled registry records.

Runtime configuration changes that affect provider routing or kill switches must be auditable.

---

## 20. Proposed Runtime Components

```txt
PlanningOrchestrator
ReconcileOrchestrator
PlanningContextBuilder
ReconcileContextBuilder
DomainSafetyClassifier
PlanningGenerationPort
ReconcileExplanationPort
PromptSchemaRegistryPort
AIOutputValidationPort
AIOperationRecorderPort
AIAvailabilityPolicy
SpringAIPlanningAdapter
SpringAIReconcileAdapter
ProviderConfigurationRegistry
```

These are responsibility boundaries, not a requirement that every name become one public interface or class.

Avoid abstraction theatre:

- one interface per capability boundary is acceptable
- wrappers that only rename a Spring method without protecting a product boundary are unnecessary
- provider-specific complexity remains behind adapters

---

## 21. Scenario Checks

### Scenario A — Planning provider returns valid-looking partial JSON

Expected:

- output rejected as PARTIAL_OUTPUT or SCHEMA_INVALID
- no reviewable subset exposed
- original user request preserved
- retry/manual fallback offered

### Scenario B — User cancels Planning while provider continues

Expected:

- operation records CANCEL_REQUESTED
- client stops waiting
- late provider output cannot mutate or silently replace current draft
- final operational state distinguishes late completion

### Scenario C — Reconcile AI is unavailable

Expected:

- deterministic FACT and RULE_MATCH remain visible
- deterministic actions remain available
- AI_EXPLANATION absent
- user receives non-alarming degraded status

### Scenario D — Imported note contains prompt injection

Expected:

- note remains untrusted content
- no permission or tool change
- output validation and allowlisted actions remain enforced
- hostile-content detection may be recorded without retaining raw text unnecessarily

### Scenario E — Provider-specific kill switch activates during request

Expected:

- in-flight behavior follows cancellation/timeout policy
- no new invocation starts
- no silent fallback unless explicitly approved
- manual/deterministic path remains available

### Scenario F — AI generates a correct proposal but state changes before confirmation

Expected:

- proposal remains only a proposal
- command path reloads current state
- stale confirmation rejected
- refreshed preview and new confirmation required

### Scenario G — Reconcile explanation invents an action

Expected:

- explanation validation rejects unknown action vocabulary
- deterministic allowed action set remains unchanged
- FACT and RULE_MATCH remain usable

### Scenario H — Crisis content enters Planning endpoint

Expected:

- ordinary generation is not invoked
- no PlanningDraft is produced from crisis content
- accepted safety flow is used
- restricted observability rules apply

---

## 22. Stabilized Proposals Pending Claude Review

- domain and application layers do not depend on Spring AI types.
- application code uses capability-specific internal ports.
- Planning and Reconcile use separate ports and context builders.
- Spring AI remains an adapter/integration framework.
- provider-specific concepts remain outside domain and client contracts.
- AI generation has no canonical mutation tools.
- Planning returns only final validated drafts, not streamed partial text.
- Reconcile is deterministic-first and may attach AI explanation later.
- cancellation never creates a usable partial draft or mutation ambiguity.
- kill switches preserve manual and deterministic product access.
- automatic cross-provider fallback is disabled by default for the pilot.
- safety classification occurs before generation and is enforced outside the model.
- all imported content remains untrusted data.
- runtime observability follows Discussion 019C without default raw-content retention.

---

## 23. Open Questions for Claude

### Q1. Are two capability-specific ports sufficient?

Review whether `PlanningGenerationPort` and `ReconcileExplanationPort` preserve the right boundary without unnecessary abstraction.

### Q2. Is Spring AI correctly contained?

Identify any proposed responsibility that still leaks Spring AI or provider concepts into application/domain layers.

### Q3. Should provider/model selection occur inside the orchestrator or a separate routing policy?

Current proposal: orchestrator requests a logical configuration; registry/routing policy resolves the actual provider route.

### Q4. Is synchronous final Planning output the right MVP decision?

Review whether content-free progress states provide enough UX without exposing partial text.

### Q5. Is hybrid Reconcile delivery coherent?

Review deterministic result first, validated AI explanation later.

### Q6. Are cancellation states sufficient?

Focus on late provider completion, request abandonment, and avoiding draft replacement races.

### Q7. Is no automatic cross-provider fallback proportionate?

Review whether a same-provider secondary model should be pre-approved for pilot resilience.

### Q8. Does the context-builder boundary enforce Discussion 018C minimization strongly enough?

Look for paths where prompt templates or adapters could fetch broader data independently.

### Q9. Is no model tool access too restrictive for MVP?

Current proposal: yes intentionally; all context is prepared by the application.

### Q10. Does Reconcile remain deterministic-first in every failure mode?

Look for accidental dependency on AI explanation before actions can be shown.

### Q11. Is the operation-scoped safety classification placement correct?

Review pre-generation classification plus post-generation policy validation.

### Q12. Could normalized failure categories lose provider information needed for debugging?

Current proposal: normalized category for application behavior plus provider-specific diagnostics only in restricted operational metadata.

### Q13. Are kill-switch checks correctly placed?

Review pre-invocation checks and behavior for in-flight requests.

### Q14. Does the proposed observability linkage duplicate or contradict Discussion 019C?

Focus on raw-content retention, crisis restrictions, and provider-neutral versioning.

### Q15. Which runtime components are unnecessary abstraction for the MVP?

Identify wrappers that do not protect a real product, privacy, reliability, or provider boundary.

### Q16. Are there hidden mutation paths through framework tool calling, callbacks, or output converters?

Review whether the prohibition is mechanically enforceable.

### Q17. Should PlanningDraft persistence occur before or after output validation?

Current proposal: create an attempt identity before invocation, persist a reviewable draft revision only after complete validation.

### Q18. Are any accepted decisions from Discussions 018A, 018C, or 019 lost in this runtime model?

Focus on confirmation, revalidation, idempotency, manual fallback, and observability access boundaries.

---

## 24. Mind Map Impact — Proposed, Do Not Apply Yet

### Product Vision

- AI remains replaceable and non-authoritative.
- product operation survives provider failure.
- provider/framework choice does not define domain behavior.
- privacy boundaries are enforced before context reaches a provider.

### MVP Core Loop

```txt
user request
→ safety/domain classification
→ scoped context builder
→ AI capability port
→ adapter/provider
→ complete validation
→ reviewable proposal or deterministic explanation
→ explicit confirmation where required
→ independent deterministic command
```

### User Flow

Add:

- content-free generation progress states
- explicit cancellation state
- manual fallback after Planning AI failure
- deterministic-first Reconcile
- AI explanation pending/unavailable state
- kill-switch degraded states

### AI Responsibilities

- interpret scoped Planning requests
- produce complete structured proposal candidates
- explain deterministic Reconcile facts
- never mutate canonical state
- never expand allowed action vocabulary

### AI Guardrails

- no provider/framework types in domain contracts
- no model mutation tools
- no unrestricted repository tools
- no partial Planning output exposed
- no free-text Reconcile context
- no silent provider fallback
- no imported-content authority
- no crisis PlanningDraft generation

### Data Events / Operational Records

Candidates for existing 019C structures:

```txt
AI_OPERATION_STARTED
AI_OPERATION_CANCEL_REQUESTED
AI_OPERATION_CANCELLED
AI_OPERATION_COMPLETED_AFTER_CANCEL_REQUEST
AI_PROVIDER_ROUTE_SELECTED
AI_KILL_SWITCH_BLOCKED_INVOCATION
AI_DEGRADED_MODE_USED
AI_CONTEXT_MANIFEST_RECORDED
```

### Traction and Guardrail Metrics

- Planning generation success rate
- Reconcile deterministic availability rate
- Reconcile explanation attachment rate
- cancellation rate
- late completion after cancellation rate
- manual fallback usage rate
- kill-switch blocked invocation count
- provider route/failure rate
- partial-output exposure rate, expected zero
- canonical mutation from AI runtime rate, expected zero

### Current Decisions

Proposed:

- internal capability ports
- Spring AI behind adapters
- synchronous validated Planning result
- hybrid deterministic-first Reconcile
- no model tools
- no automatic cross-provider fallback

### Open Questions

- same-provider secondary model fallback
- exact cancellation transport
- progress delivery mechanism
- minimum number of runtime interfaces
- runtime configuration registry implementation

---

## 25. Affected Formal Documents — Record Only

After closure, update or create:

- AI runtime architecture specification
- Spring AI adapter ADR
- provider abstraction and routing ADR
- Planning orchestration sequence
- Reconcile orchestration sequence
- context minimization implementation contract
- kill-switch and degraded-mode specification
- cancellation semantics contract
- runtime threat model
- API contracts in Discussion 020B
- reliability and cost controls in Discussion 020C
- validation and release tests in Discussion 021
- implementation sequence in Discussion 022

---

## 26. Structured Review Request for Claude

Review this proposal for architectural coherence, safety, privacy, provider replaceability, and proportional MVP complexity.

For every finding, provide:

```txt
1. Finding ID
2. severity: blocking, important, or minor
3. affected section
4. concrete runtime, privacy, or failure scenario
5. why the proposal is unsafe, contradictory, leaky, or overengineered
6. smallest coherent correction
7. affected earlier or later discussion
```

Focus especially on:

- Spring AI leakage into application/domain layers
- accidental model mutation authority
- Planning versus Reconcile capability separation
- context minimization enforcement
- sync versus hybrid delivery
- cancellation and late completion races
- kill-switch behavior
- provider fallback risk
- hostile imported content
- crisis routing
- observability and raw-content boundaries
- unnecessary abstraction

Do not expand into endpoint naming, final DTO schemas, exact timeout numbers, database vendor syntax, frontend styling, or implementation sequencing.

This discussion remains open until Claude reviews the proposal and Mahdi resolves accepted findings.
