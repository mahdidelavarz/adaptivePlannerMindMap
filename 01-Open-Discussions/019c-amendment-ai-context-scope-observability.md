# Discussion 019C Amendment — AI Context Scope Observability

## Status

Accepted as a narrow amendment required by Discussion 020A.

This amendment extends, but does not otherwise reopen:

- [[01-Open-Discussions/019c-final-events-ai-observability-and-retention-resolution]]
- [[01-Open-Discussions/020a-final-ai-runtime-boundaries-and-orchestration-resolution]]

Where this amendment is more specific, it is authoritative for AI context-scope observability.

---

## 1. Reason for Amendment

Discussion 020A requires the runtime to prove which context-building policy produced an AI request without duplicating the actual prompt or canonical entity content.

The final Discussion 019C `AIOperation` model did not previously include:

- `contextBuilderVersion`
- an explicit Context Scope Manifest

This amendment adds those observability requirements explicitly rather than treating them as an implied schema change.

---

## 2. AIOperation Addition

`AIOperation` adds:

```txt
contextBuilderKey
contextBuilderVersion
contextScopeManifestId?
```

The logical key and version identify the deterministic context builder used for the operation.

They must not contain provider-specific implementation names unless those names are already part of provider-neutral runtime configuration metadata.

---

## 3. AIContextScopeManifest

Proposed linked record:

```txt
AIContextScopeManifest
- id
- aiOperationId
- builderKey
- builderVersion
- operationType
- includedContextCategories[]
- includedEntityTypes[]
- includedEntityCount
- includedFieldGroups[]
- excludedSensitiveCategories[]
- freeTextIncluded
- importedContentIncluded
- createdAt
```

The manifest records scope metadata only.

It must not duplicate:

- raw prompt text
- raw response text
- entity descriptions
- user-authored notes
- imported document content
- canonical entity snapshots
- authentication or secret material

```txt
Context Scope Manifest
→ what categories were included
≠ the actual included content
```

---

## 4. Required Invariants

- Every Planning and Reconcile AI operation records a context builder key and version.
- Reconcile MVP operations record `freeTextIncluded = false`.
- Reconcile MVP operations record `importedContentIncluded = false`.
- Planning operations record only the categories actually supplied.
- The manifest cannot grant permission to send data; it only records the result of an already-authorized context-building decision.
- Missing manifest data must not cause the runtime to silently broaden context.
- Context scope metadata inherits the access and retention boundaries of AI operational metadata under Discussion 019C.
- Raw content retention rules remain unchanged.

---

## 5. Retention and Access

`AIContextScopeManifest` uses the same default retention class and access boundary as `AIOperation` metadata.

It is available to restricted engineering and safety review roles for:

- privacy verification
- prompt/context regression analysis
- incident investigation
- version comparison

It is not an ordinary product analytics dimension for reconstructing sensitive user behavior.

---

## 6. Affected Later Work

### Discussion 020B

API contracts should not expose the full manifest to ordinary clients. User-facing privacy explanations may use a separately curated summary.

### Discussion 020C

Validation tests must verify:

- builder version is recorded
- scope metadata matches the actual constructed DTO categories
- Reconcile free-text flags remain false
- sensitive categories are never silently included

### Discussion 021

Add regression tests for context-scope enforcement and manifest correctness.

---

## 7. Closure

This amendment is accepted and closed.

It adds explicit context-scope observability without reopening the four-layer persistence model, raw-content policy, retention model, or canonical-versus-observability boundary accepted in Discussion 019C.
