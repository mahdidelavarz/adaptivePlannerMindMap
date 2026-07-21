# Discussion 022 — MVP Baseline Reconciliation, Mind Map Migration, and Implementation Plan

## Status

Open for GPT × Claude review.

## Purpose

Discussion 022 is the final planning discussion before implementation planning becomes authoritative.

Its purpose is not merely to order engineering milestones. It must first reconcile the legacy non-AI MVP decisions, establish one authoritative AI-native MVP baseline, migrate all accepted decisions into the shared Mind Map, verify that the Mind Map and accepted discussion set agree, and only then derive the implementation sequence.

```txt
legacy decisions
→ decision-level audit
→ supersession and amendment resolution
→ final MVP baseline
→ Mind Map migration
→ consistency verification
→ vertical implementation plan
→ pilot readiness
```

An implementation roadmap built before these steps would be based on a stale product map and ambiguous source documents.

---

## 1. Governing Rules

1. Discussion 022 may not silently change accepted product behavior from Discussions 010–021.
2. Any required behavior change must reopen or amend the source discussion explicitly.
3. Legacy Discussions 001–009 are not automatically valid and are not automatically obsolete.
4. Legacy review must occur at decision level, not only document level.
5. A legacy document may contain retained, amended, superseded, and unresolved decisions simultaneously.
6. The shared Mind Map is not authoritative until every accepted change from Discussions 010–021 is applied and verified.
7. Writing `Mind Map Impact` inside a Markdown discussion does not count as applying the change to the Mind Map.
8. No implementation milestone becomes authoritative before the final MVP baseline and Mind Map migration are approved.
9. Events, observability, safety, privacy, idempotency, and validation are cross-cutting requirements from the first vertical slice, not end-stage cleanup.
10. Pilot readiness and public release readiness are different gates.
11. Schedule estimates are downstream of scope, dependencies, ownership, and team capacity; Discussion 022 defines sequence and gates, not decorative calendar optimism.

---

## 2. Source-of-Truth Hierarchy

When sources disagree, use this precedence order:

1. accepted final resolutions and explicit amendments,
2. accepted source discussions without a separate final resolution,
3. verified current Mind Map nodes linked to accepted discussions,
4. legacy discussions not yet audited,
5. old implementation plans, sketches, and historical notes.

The current Mind Map may not override an accepted resolution merely because the map is easier to see.

Each final baseline decision must identify exactly one authoritative source.

---

## 3. Workstream A — Legacy Discussion Audit

### 3.1 Audit Scope

Audit Discussions 001–009.

Discussions 001–008 were primarily created for the previous non-AI MVP. Discussion 009 must also be audited rather than excluded by assumption, because it may contain transitional decisions or dependencies spanning both product versions.

### 3.2 Decision-Level Status Vocabulary

Each material decision receives exactly one status:

- `RETAINED`
- `AMENDED`
- `PARTIALLY_SUPERSEDED`
- `FULLY_SUPERSEDED`
- `REQUIRES_REVIEW`
- `NON_NORMATIVE_HISTORY`

Definitions:

#### RETAINED

The decision remains valid without behavioral change.

#### AMENDED

The decision remains partially valid but must be read together with a named amendment or later resolution.

#### PARTIALLY_SUPERSEDED

Some parts remain valid, while named parts are replaced by later decisions.

#### FULLY_SUPERSEDED

The decision no longer governs current product behavior.

#### REQUIRES_REVIEW

No later decision clearly resolves whether the legacy rule remains valid.

#### NON_NORMATIVE_HISTORY

The content may explain product evolution but must not guide implementation.

### 3.3 Required Legacy Audit Record

For every material legacy decision, record:

```txt
legacyDiscussionId
legacyDecisionId
summary
originalScope
status
retainedMeaning
supersededMeaning
replacementSource
affectedMindMapNodes
implementationImpact
openConflict
reviewOwner
```

### 3.4 Document-Level Summary

After decision-level classification, each legacy discussion receives a document summary:

```txt
fully retained
retained with amendments
partially superseded
fully superseded
requires additional resolution
```

This summary does not replace decision-level records.

### 3.5 Supersession Safety Rules

- No legacy decision is superseded merely because it does not mention AI.
- No legacy decision remains valid merely because no later document repeats it.
- Absence of conflict is not proof of continued relevance.
- Product terminology changes must be traced explicitly.
- Data model, lifecycle, ownership, temporal, safety, and deletion semantics require explicit comparison against Discussions 010–021.
- A legacy implementation shortcut may be discarded even when the product behavior it supported is retained.

### 3.6 Required Output A

Produce a **Legacy Decision Audit and Supersession Matrix** covering Discussions 001–009.

The matrix must make it impossible for an implementer to mistake historical behavior for current behavior.

---

## 4. Workstream B — Accepted Decision Inventory

### 4.1 Inventory Scope

Inventory Discussions 010–021 and all associated final resolutions or amendments.

For each accepted discussion, extract:

```txt
authoritativeFile
acceptedDecisionIds
newTerms
changedTerms
newInvariants
changedUserFlows
changedDataConcepts
newEvents
newGuardrails
newMetrics
newOpenQuestions
MindMapImpactItems
specificationsOrADRsRequired
```

### 4.2 Duplicate and Conflict Detection

The inventory must detect:

- two documents claiming authority over the same behavior,
- later documents depending on earlier unresolved terms,
- inconsistent status names,
- renamed entities without migration mapping,
- contradictory user flows,
- missing event support for accepted metrics,
- missing implementation artifacts promised by final resolutions,
- references to discussions or release gates that do not exist or were not carried forward.

### 4.3 Required Output B

Produce an **Accepted Decision Inventory** for Discussions 010–021 with links to authoritative files and every extracted Mind Map change.

---

## 5. Workstream C — Final MVP Baseline

The final MVP baseline is the smallest complete product scope that honestly implements:

```txt
Plan → Execute → Adapt
```

with the accepted AI-native safety, authority, and validation boundaries.

### 5.1 Baseline Categories

The baseline must explicitly define:

- target pilot user,
- core product promise,
- canonical entities,
- ownership and lifecycle rules,
- manual planning capabilities,
- AI-assisted Planning responsibilities,
- Today execution behavior,
- Routine behavior,
- deterministic Reconcile behavior,
- AI-assisted Reconcile behavior,
- confirmation and mutation boundaries,
- safety and crisis behavior,
- data events and observability,
- validation metrics and decision gates,
- pilot constraints,
- excluded functionality.

### 5.2 Baseline Status Vocabulary

Every candidate feature or behavior receives one status:

- `MVP_REQUIRED`
- `PILOT_REQUIRED_NON_PRODUCT`
- `POST_PILOT`
- `REMOVED`
- `UNRESOLVED`

`PILOT_REQUIRED_NON_PRODUCT` includes instrumentation, safety evidence, operational tooling, runbooks, and reviewer workflows that may not be visible as product features but are mandatory for pilot launch.

### 5.3 Thesis-Preserving Scope Rule

A scope cut is valid only when the remaining product still demonstrates all three parts of the loop:

- users can form a meaningful plan,
- users can execute and record reality,
- users can adapt when reality diverges.

The following cuts are presumed thesis-breaking unless a source discussion is reopened:

- removing manual planning fallback,
- removing Today execution,
- removing deterministic Reconcile,
- letting AI mutate canonical state directly,
- removing confirmation boundaries,
- removing crisis fail-closed behavior,
- removing events needed for Discussion 021 gates,
- shipping AI Planning without an Adapt loop,
- shipping Reconcile without manual escape.

### 5.4 Required Output C

Produce a **Final MVP Baseline Matrix** mapping every retained capability to its source discussion, map section, implementation milestone, and pilot gate.

---

## 6. Workstream D — Mind Map Migration

### 6.1 Migration Principle

The shared Mind Map must become a verified projection of accepted product decisions, not a parallel product specification.

### 6.2 Current Known Gap

Discussions 010–021 contain written `Mind Map Impact` sections, but those changes have not yet been applied to the actual map.

Therefore, the current map must be treated as stale until migration is complete.

### 6.3 Migration Record

Every map change must use this record:

```txt
migrationItemId
sourceDiscussion
sourceDecision
mapSection
targetNode
changeType
oldContent
newContent
linkedDependencies
owner
reviewer
status
evidenceLink
```

Allowed `changeType` values:

- `CREATE_NODE`
- `UPDATE_NODE`
- `MOVE_NODE`
- `DELETE_NODE`
- `MARK_SUPERSEDED`
- `CREATE_RELATION`
- `DELETE_RELATION`
- `ADD_REFERENCE`

### 6.4 Minimum Map Sections

Migration must review and update at least:

1. Product Vision
2. MVP Core Loop
3. User Flow
4. AI Responsibilities
5. AI Guardrails
6. Data Model / Domain Concepts
7. Data Events
8. Traction Metrics
9. Current Decisions
10. Open Questions
11. Implementation / Delivery Plan
12. References and Study Needs

### 6.5 Expected Discussion-to-Map Coverage

At minimum, verify these categories:

- product and domain semantics from Discussions 010–018,
- transaction, concurrency, idempotency, events, and observability from Discussion 019,
- AI runtime, API contracts, structured output, reliability, cost, and kill switches from Discussion 020,
- hypotheses, metrics, thresholds, crisis release gate, and decision gates from Discussion 021,
- final baseline and implementation sequencing from Discussion 022.

### 6.6 Historical Node Policy

Old nodes may be preserved for history only when clearly marked:

```txt
SUPERSEDED — DO NOT IMPLEMENT
```

A visually present but silently obsolete node is forbidden.

### 6.7 Migration Completeness Gate

Mind Map migration passes only when:

- every accepted `Mind Map Impact` item has a migration record,
- every migration record has been applied or explicitly rejected with reason,
- every changed node links back to an authoritative discussion,
- no unresolved contradiction remains hidden inside an active node,
- legacy nodes are retained, amended, or visibly superseded,
- designer, frontend, backend, and product reviewers can identify the same MVP baseline from the map,
- a decision-to-node traceability check passes.

### 6.8 Required Output D

Produce and execute a **Mind Map Migration Ledger**. The ledger alone is not completion; the actual FigJam board must be changed and reviewed.

---

## 7. Workstream E — Mind Map Verification

After migration, perform a structured consistency review.

### 7.1 Verification Questions

For every active node:

- Which accepted decision authorizes it?
- Is the terminology current?
- Is the behavior complete or misleadingly simplified?
- Does it contradict another active node?
- Does it omit a safety, authority, or failure condition?
- Is it implementation guidance, product behavior, historical context, or an open question?

### 7.2 Cross-Role Walkthrough

The designer, frontend owner, and backend owner independently explain from the map:

```txt
user planning flow
AI proposal flow
confirmation and command flow
Today execution flow
Reconcile flow
crisis fallback flow
pilot measurement flow
```

Material disagreement means verification fails.

### 7.3 Baseline Snapshot

After approval, capture:

- board version or dated snapshot,
- migration ledger version,
- authoritative discussion index,
- unresolved-question list,
- reviewer sign-offs.

This snapshot becomes the planning input for implementation milestones.

---

## 8. Workstream F — Implementation Strategy

### 8.1 Vertical Slice Rule

Each milestone must deliver an end-to-end behavior across the necessary layers:

```txt
UX/design
→ frontend
→ API contract
→ domain behavior
→ persistence
→ events/outbox
→ observability
→ tests
```

Layer-only phases are permitted only for narrowly bounded enabling work with explicit consumers in the next milestone.

### 8.2 Smallest End-to-End Slice

Candidate first product slice:

```txt
manually create one Task
→ task appears in Today
→ complete Task
→ deterministic canonical state changes
→ event and outbox records persist
→ frontend reflects CommandResult
```

This slice must include auth/ownership, versioning, failure handling, and basic observability.

### 8.3 Proposed Milestone Sequence

#### M0 — Reconciliation and Map Baseline

Deliverables:

- legacy audit,
- accepted decision inventory,
- final MVP baseline,
- migrated and verified Mind Map,
- authoritative discussion index.

Exit gate:

- Workstreams A–E pass,
- no implementation-blocking contradiction remains.

#### M1 — Delivery and Domain Foundation

Deliverables:

- repository and environment conventions,
- authentication and ownership baseline,
- canonical identifiers and version fields,
- transaction boundary conventions,
- transactional outbox foundation,
- event envelope and observability baseline,
- API error envelope,
- test infrastructure,
- migration strategy.

Exit gate:

- foundation supports M2 without introducing mock domain behavior that must later be discarded.

#### M2 — Manual Plan → Today → Complete

Deliverables:

- manual Task creation and editing,
- Today projection,
- completion flow,
- CommandResult boundary,
- idempotency and stale-version handling,
- events and pilot-compatible instrumentation for this slice.

Exit gate:

- complete end-to-end slice passes product, contract, concurrency, accessibility, and observability tests.

#### M3 — Routine Generation and Execution

Deliverables:

- Routine definition,
- occurrence generation,
- Today integration,
- completion/skip semantics,
- timezone and recurrence handling,
- relevant events and tests.

Exit gate:

- occurrence generation and execution invariants pass deterministic tests across timezone and boundary cases.

#### M4 — PlanningDraft Flow with Deterministic Mock Provider

Deliverables:

- PlanningAttempt lifecycle,
- reviewable PlanningDraft lifecycle,
- immutable revisions,
- server-authoritative preview,
- edit and confirmation flow,
- CommandResult linkage,
- cancellation and stale-result handling,
- deterministic mock AI adapter producing fixed valid/invalid fixtures.

Exit gate:

- complete API/frontend/domain flow works before real provider integration,
- invalid or partial fixture output never creates a reviewable resource.

#### M5 — Real AI Planning Runtime

Deliverables:

- runtime ports from 020A,
- bounded context builder,
- pinned artifact bundle,
- structured-output pipeline from 020C,
- retry, timeout, rate limit, circuit breaker, spend cap, and kill switches,
- safety classification and crisis fallback integration,
- operational dashboards and alerts.

Exit gate:

- all 020A–020C architecture and adversarial tests pass,
- manual planning remains fully available,
- crisis safety readiness preconditions relevant to Planning pass.

#### M6 — Deterministic Reconcile

Deliverables:

- deterministic facts and rule matches,
- severity classifier,
- Light/Medium/Recovery behavior,
- manual resolution paths,
- degraded and unavailable states,
- event and metric support.

Exit gate:

- Reconcile produces no AI dependency for facts,
- manual escape succeeds for all designed failure states,
- deterministic failure never falls back to raw-data AI summary.

#### M7 — AI Reconcile Explanation

Deliverables:

- explanation-only AI context,
- no free-text raw data where prohibited,
- recommendation review and disposition,
- confirmation and CommandResult boundaries,
- degraded facts-only mode,
- cancellation and validation behavior.

Exit gate:

- AI cannot create canonical facts or mutation authority,
- facts-only Reconcile remains usable when AI is unavailable.

#### M8 — Validation Instrumentation and Pilot Operations Completion

Deliverables:

- complete Discussion 021 metric mapping,
- denominator verification,
- severity segmentation,
- qualitative research workflow,
- regret attribution workflow,
- trust quiz and behavioral cross-check,
- pilot dashboards,
- incident and rollback runbooks,
- privacy and retention verification.

Exit gate:

- every pilot metric can be reproduced from accepted events,
- missing-data behavior is known,
- no restricted crisis data enters ordinary analytics.

#### M9 — Pilot Readiness and Controlled Rollout

Deliverables:

- crisis safety readiness sign-off,
- security and privacy review,
- provider spend cap verification,
- kill-switch drill,
- backup/manual paths,
- support procedures,
- pilot cohort and consent readiness,
- frozen threshold package,
- rollout and rollback checklist.

Exit gate:

- every hard gate in Discussion 021 passes,
- unresolved pilot risks have explicit owners and accepted disposition.

### 8.4 Cross-Cutting Requirements

These begin no later than M1 and evolve with each slice:

- authorization,
- accessibility,
- event instrumentation,
- logs and traces,
- privacy and retention,
- deterministic test fixtures,
- schema and migration discipline,
- idempotency,
- cancellation,
- failure-state UX,
- security review,
- documentation and traceability.

They may not be postponed to M8 or M9.

---

## 9. Team Ownership Model

Assumed core team:

- Product/Frontend owner
- Backend owner
- Product designer

Additional review roles may be part-time or external:

- Safety/Policy reviewer
- Security/Privacy reviewer
- Pilot research owner

### 9.1 Responsibility Rules

Every milestone must identify:

```txt
accountableOwner
frontendOwner
backendOwner
designOwner
reviewers
dependencies
handoffArtifacts
exitGateApprovers
```

### 9.2 Parallel Work

Parallel work is allowed when contracts are explicit.

Examples:

- design may define review, confirmation, failure, degraded, and cancellation states while backend implements domain handlers,
- frontend may build against versioned mock contracts while backend implements the same accepted schema,
- backend may build runtime adapters while frontend completes PlanningDraft state machines using deterministic mock attempts,
- pilot instrumentation design may proceed alongside feature implementation, but event schemas must be accepted before feature code is considered complete.

Parallelism must not create competing contract versions.

---

## 10. Contract and Migration Freeze Policy

### 10.1 Freeze Levels

Use these levels:

- `DRAFT`
- `SLICE_LOCKED`
- `PILOT_LOCKED`
- `POST_PILOT_CHANGE`

### 10.2 Slice Lock

Before frontend/backend integration for a milestone:

- resource schemas,
- command contracts,
- error codes,
- event names and required fields,
- version semantics,
- migration assumptions

must be `SLICE_LOCKED`.

Changes require explicit amendment and coordinated consumer updates.

### 10.3 Pilot Lock

Before M9:

- externally observed API behavior,
- pilot event semantics,
- metric denominators,
- severity classifier version,
- threshold package,
- safety policy and crisis copy,
- runtime artifact versions

must be `PILOT_LOCKED`.

Emergency safety changes remain allowed but must be audited and included in pilot analysis.

---

## 11. Reusable, Amended, and Superseded Work

Existing implementation work must be classified separately from product decisions.

Statuses:

- `REUSE_AS_IS`
- `REUSE_WITH_TESTS`
- `AMEND`
- `MIGRATE`
- `REPLACE`
- `REMOVE`
- `UNKNOWN_UNTIL_AUDIT`

Evaluation criteria:

- compatibility with accepted domain model,
- ownership and authorization correctness,
- version and concurrency behavior,
- API and error-envelope compatibility,
- event and observability support,
- AI authority boundaries,
- privacy and retention compliance,
- testability,
- cost of adapting versus replacing.

A reusable UI component does not imply its old flow remains valid. A reusable persistence table does not imply its old lifecycle semantics remain valid.

Required output: **Implementation Reuse and Supersession Matrix**.

---

## 12. Test Strategy by Layer and Slice

Every milestone defines required tests in these categories:

- domain invariants,
- command and transaction behavior,
- concurrency and idempotency,
- API contract,
- frontend state machine,
- accessibility,
- persistence and migrations,
- event/outbox delivery,
- observability,
- privacy and retention,
- safety and adversarial behavior,
- end-to-end user flow,
- rollback and kill-switch behavior where applicable.

### 12.1 AI-Specific Test Families

- structured-output conformance,
- invalid and partial output rejection,
- temporary-reference graph validation,
- prompt-injection resistance,
- context-scope enforcement,
- cancellation race,
- timeout and retry limits,
- no hidden retry,
- spend-cap enforcement,
- provider-unavailable degraded mode,
- no direct mutation authority,
- crisis routing and zero proposal leakage.

### 12.2 Pilot Metric Test Families

- exact numerator and denominator reproduction,
- exclusion handling,
- duplicate attempt handling,
- user-level weighting,
- severity segmentation,
- missing-data classification,
- regret attribution categories,
- trust quiz/behavior mismatch reporting,
- restricted-event exclusion from ordinary analytics.

---

## 13. Scope-Cut Ladder

Scope cuts must be ordered from safest to most thesis-threatening.

### Level 1 — Presentation and Convenience Cuts

Examples:

- non-essential visual polish,
- secondary filters,
- optional personalization,
- advanced export,
- non-critical dashboards.

### Level 2 — Breadth Cuts

Examples:

- reduce supported entity combinations,
- limit Routine recurrence patterns,
- limit AI Planning operation families,
- pilot only one locale while preserving safe fallback behavior,
- limit Recovery complexity while retaining deterministic manual resolution.

### Level 3 — Capability Deferral

Examples requiring explicit validation impact review:

- defer AI Reconcile explanation while keeping deterministic Reconcile,
- pilot AI Planning for a narrower planning scenario,
- defer lower-priority analytics not required by Discussion 021.

### Level 4 — Thesis-Breaking Cuts

Forbidden without reopening source discussions:

- remove Today execution,
- remove Adapt/Reconcile,
- remove manual fallback,
- remove confirmation and deterministic command boundaries,
- weaken crisis or safety gates,
- remove events required to judge the pilot,
- permit direct AI mutation,
- collapse accepted and applied states.

Every proposed cut records:

```txt
cutId
removedScope
reason
savedCostOrTime
hypothesisImpact
metricImpact
safetyImpact
mapChanges
sourceDiscussionsToReopen
approvers
```

---

## 14. Pilot Readiness versus Release Readiness

### 14.1 Pilot Readiness

Pilot readiness permits a bounded, consented, monitored cohort under accepted operational constraints.

It requires:

- final MVP baseline implemented,
- Mind Map and implementation traceability current,
- all Discussion 021 hard gates passing,
- crisis safety readiness passing,
- manual and degraded paths working,
- instrumentation and analysis package locked,
- support, incident, kill-switch, rollback, and spend-cap procedures tested,
- known limitations disclosed to participants,
- pilot-specific privacy and retention controls verified.

### 14.2 Release Readiness

Public or broader release requires additional evidence not decided merely by completing implementation:

- pilot decision gates support continuation,
- operational reliability is sustained,
- security/privacy findings are resolved,
- crisis and safety performance remains acceptable,
- support and cost models are viable,
- known pilot-only constraints are removed or explicitly retained,
- rollout strategy is approved.

Passing implementation milestones does not automatically pass release readiness.

---

## 15. Required Artifacts

Discussion 022 must produce or require:

1. Legacy Decision Audit and Supersession Matrix
2. Accepted Decision Inventory
3. Authoritative Discussion Index
4. Final MVP Baseline Matrix
5. Mind Map Migration Ledger
6. Updated and verified FigJam Mind Map
7. Implementation Reuse and Supersession Matrix
8. Dependency Graph
9. Milestone and Exit Gate Plan
10. Ownership Matrix
11. Contract Freeze Register
12. Test Coverage Matrix
13. Scope-Cut Register
14. Pilot Readiness Checklist
15. Release Readiness Checklist
16. Risk and Contingency Register

---

## 16. Resolution Sequence

Discussion 022 closes only after this order is completed:

```txt
A. audit Discussions 001–009
B. inventory Discussions 010–021
C. resolve conflicts and produce final MVP baseline
D. create migration ledger
E. update actual Mind Map
F. verify map consistency across roles
G. classify reusable and superseded implementation work
H. approve milestone sequence, ownership, and exit gates
I. approve pilot-readiness checklist
J. publish final Discussion 022 resolution
```

No step may be marked complete solely because a future action was documented.

---

## 17. Questions for Claude Review

1. Is decision-level auditing of Discussions 001–009 sufficient, or does any legacy topic require a dedicated reopening before baseline approval?
2. Is Discussion 009 correctly included in the legacy audit scope?
3. Does the source-of-truth hierarchy safely resolve disagreement between final resolutions, open discussions, and the stale Mind Map?
4. Are the legacy status categories mutually clear and implementation-safe?
5. What failure scenario could allow a partially superseded legacy decision to remain active accidentally?
6. Does the Accepted Decision Inventory capture all ways Discussions 010–021 may conflict or depend on missing artifacts?
7. Is the final MVP baseline definition sufficient to prevent implementation from omitting part of `Plan → Execute → Adapt`?
8. Which listed scope cuts are incorrectly classified as safe, conditional, or thesis-breaking?
9. Is the Mind Map Migration Ledger sufficient for traceability, or does it need node IDs, board version IDs, or immutable snapshots as mandatory fields?
10. Does the migration completeness gate prove that decisions were actually applied rather than merely listed?
11. Is the cross-role walkthrough an adequate consistency check, and what concrete mismatch should fail it?
12. Is M0 correctly treated as a real blocking milestone rather than documentation overhead?
13. Is the M1–M9 sequence genuinely vertical, or does it still hide layer-first implementation?
14. Should Routine precede PlanningDraft, or would another order reduce rework without weakening the complete loop?
15. Is deterministic mock AI introduced at the correct milestone and with sufficient fidelity?
16. Are events, observability, privacy, safety, and idempotency introduced early enough in every slice?
17. Are contract freeze levels and change procedures sufficient for parallel frontend/backend/design work?
18. Does the reuse matrix safely separate reusable code from superseded behavior?
19. Are pilot readiness and release readiness separated strongly enough?
20. Which mandatory tests or operational drills are still missing before real-user exposure?
21. Could Discussion 022 close while any accepted Mind Map Impact item remains unapplied? The intended answer is no; identify loopholes.
22. Which dependencies on Discussions 019–021 are not represented clearly enough in the milestones?
23. Does the crisis safety gate appear at the correct points in implementation and pilot readiness?
24. Is any new product behavior accidentally introduced by this plan rather than traced to Discussions 010–021?

---

## 18. Expected Claude Finding Format

```txt
Finding ID
Severity: Blocking | Important | Minor
Affected section
Concrete failure scenario
Why the current rule is incomplete or contradictory
Smallest coherent correction
Affected source discussions or Mind Map sections
```

Claude should prioritize:

- hidden contradictions between legacy and accepted decisions,
- incomplete supersession rules,
- migration steps that can be marked complete without changing FigJam,
- milestone sequencing that creates avoidable rework,
- missing cross-cutting safety or observability work,
- scope cuts that make the product thesis dishonest,
- pilot-readiness gaps.

---

## 19. Expected Final Resolution Outputs

The final resolution must contain:

- accepted audit procedure,
- accepted supersession matrix,
- accepted authoritative source index,
- accepted final MVP baseline,
- completed Mind Map migration evidence,
- verified map snapshot,
- accepted implementation milestone sequence,
- ownership and dependency model,
- exit gates,
- reusable/superseded implementation classification,
- test strategy,
- scope-cut ladder,
- pilot-readiness and release-readiness gates,
- remaining open questions,
- affected specifications and ADRs.

Discussion 022 is not resolved merely by approving this plan. It is resolved when the reconciliation and Mind Map migration work has actually been completed and the resulting implementation plan is accepted.