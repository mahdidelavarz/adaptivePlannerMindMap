# Start Here — AI-Native MVP Reconciliation

## Current state

The old Reconcile-first Phase 1 MVP and its implementation-ready framing are deprecated.

The accepted AI-native direction has been consolidated, but the repository-wide migration is still in progress. Do not begin implementation from the old Canvas, Phase 1 plan, ADRs, flows, or specifications.

```txt
Accepted product decisions       Closed Discussions 010–021
Legacy compatibility decisions  001–008 surviving-decisions record
Decision index                   Complete
AI-native MVP baseline           Complete
Mind Map                         Stale; migration pending
Discussion 022                   Open
Implementation plan              Not yet authoritative
```

## Required reading order

### Orientation

1. [[02-Decisions/accepted-decision-inventory-001-021]] — authoritative discussion index, decision IDs, ownership boundaries, and conflict resolutions
2. [[04-Specs/ai-native-mvp-baseline]] — MVP-required, pilot-required, post-pilot, removed, and unresolved scope
3. [[01-Open-Discussions/022-updated-mvp-implementation-plan]] — migration, verification, sequencing, and readiness gates

### Mind Map migration

4. [[00-Canvas/Planner-Mindmap-Migration-Ledger]] — prepared migration operations and traceability
5. [[00-Canvas/Planner-Mindmap.canvas]] — inspect only as the stale migration input until the ledger is applied and verified
6. [[02-Decisions/repository-artifact-migration-ledger]] — classifications, required reviewers, migration gates, and unresolved-configuration ownership for the rest of the vault

### Detailed behavior

Read the relevant authoritative closed discussion directly. Use the accepted-decision inventory to find the owner of each subject:

- product direction and scope: Discussions 010–011;
- product and temporal models: Discussions 012–012A;
- AI Planning flow and output: Discussions 013–014A;
- Today, Task, and Routine execution: Discussions 015–015B;
- deterministic and AI-assisted Reconcile: Discussions 016–017A;
- authority, reversibility, privacy, hostile input, and crisis safety: Discussions 018–018A;
- canonical persistence, transactions, events, and retention: Discussions 019A–019C;
- runtime, API/frontend state, structured output, reliability, and cost: Discussions 020A–020C;
- validation, metrics, and decision gates: Discussion 021.

For narrowly retained authentication, API, research, development, and pilot-operation compatibility decisions, read [[01-Closed-Discussions/001-008-legacy-surviving-decisions]]. Do not reconstruct legacy authority from deleted discussions or old artifacts.

## Product invariant

The MVP must preserve the complete loop:

```txt
Plan → Execute → Adapt
```

The minimum interpretation is:

1. a user can manually plan or enter a bounded AI Planning flow;
2. AI output becomes a validated, reviewable proposal—not canonical state;
3. the user reviews and confirms visible consequences;
4. deterministic services revalidate and commit the command;
5. Today supports execution and records reality;
6. deterministic Reconcile derives facts and severity;
7. optional bounded AI may explain or organize permitted recommendations;
8. adaptation requires preview, confirmation, deterministic mutation, and events.

Manual escape, confirmation boundaries, crisis fail-closed behavior, access control, idempotency, events, observability, and pilot evidence are cross-cutting requirements rather than later cleanup.

## What is not safe to use yet

Unless Discussion 022 explicitly reconciles and adopts them, do not treat these as current implementation authority:

- [[02-Decisions/ADR-001-reconcile-on-open]]
- [[02-Decisions/ADR-002-phase-1-technical-foundation]]
- [[02-Decisions/ADR-003-phase-0-closure]]
- [[05-Implementation/phase-1-plan]]
- [[05-Flows/mvp-core-loop]]
- the older Phase 1 specifications under `04-Specs/`
- the current visible structure of [[00-Canvas/Planner-Mindmap.canvas]]

These files may contain reusable technical material, but reusable code or design does not reactivate deprecated product behavior.

## Next authorized documentation step

The next repository-wide step is to review and approve the Mind Map migration ledger, apply it only if the Canvas hash still matches, and then verify the migrated map across Product, Design, Frontend, Backend, Safety, and Research perspectives.

After the map passes verification, continue Discussion 022’s remaining workstreams:

```txt
classify reusable versus superseded implementation material
→ approve dependencies, milestones, ownership, and exit gates
→ approve pilot-readiness requirements
→ publish the final Discussion 022 resolution
```

Implementation is ready only after those gates make the new plan authoritative.

## Change rule

Use the accepted-decision inventory to identify the owning discussion before changing behavior.

- A documentation migration may clarify or project accepted decisions.
- It may not silently introduce new behavior.
- A genuine behavior change must reopen or explicitly amend the authoritative source discussion.
