# Adaptive Planner Mind Map

This Obsidian vault contains the product decisions, architecture, research, migration records, and implementation planning for Adaptive Planner.

The former Reconcile-first Phase 1 MVP is deprecated. The accepted product direction is now the AI-native MVP defined by the closed Discussions 010–021 and reconciled through Discussion 022.

Start with:

1. [[00-START-HERE]]
2. [[02-Decisions/accepted-decision-inventory-001-021]]
3. [[04-Specs/ai-native-mvp-baseline]]
4. [[01-Open-Discussions/022-updated-mvp-implementation-plan]]
5. [[00-Canvas/Planner-Mindmap-Migration-Ledger]]
6. [[02-Decisions/repository-artifact-migration-ledger]]

## Current state

```txt
Legacy decision reconciliation                 COMPLETE
Accepted-decision inventory                    COMPLETE
AI-native MVP baseline                         COMPLETE
Mind Map migration ledger                      PREPARED, NOT APPROVED
Repository Mind Map migration                  NOT APPLIED
Cross-role Mind Map verification               PENDING
Authoritative implementation plan              PENDING
Implementation readiness                       NOT YET APPROVED
```

The current [[00-Canvas/Planner-Mindmap.canvas]] still represents the deprecated MVP. It is a migration target, not an authoritative description of the product.

## Current product frame

The AI-native MVP must demonstrate the complete loop:

```txt
Plan → Execute → Adapt
```

- Planning is the first-value engine.
- Today provides execution and reality evidence.
- deterministic Reconcile identifies and structures divergence.
- bounded AI may propose plans and explain permitted evidence.
- AI never owns authority or directly mutates canonical state.
- the user reviews and explicitly confirms consequential changes.
- deterministic application services revalidate and commit changes atomically.
- manual paths, safety boundaries, observability, and pilot evidence are required from the relevant first slice.

Detailed behavior belongs to the authoritative closed discussions and the accepted-decision inventory; this README is only an entry point.

## Source-of-truth hierarchy

When documents disagree, use this order:

1. authoritative closed Discussions 010–021 and their explicit ownership boundaries;
2. [[01-Closed-Discussions/001-008-legacy-surviving-decisions]] for narrowly retained compatibility decisions only;
3. verified nodes in [[00-Canvas/Planner-Mindmap.canvas]] **after** the migration ledger is applied and verification passes;
4. formal specifications and ADRs explicitly reconciled and adopted by Discussion 022;
5. older specifications, ADRs, implementation plans, flows, sketches, and Git history as non-normative evidence only.

The consolidated index is [[02-Decisions/accepted-decision-inventory-001-021]]. The scope projection is [[04-Specs/ai-native-mvp-baseline]].

## Migration warning

Files outside the authoritative set have not all been reconciled yet. In particular, the following may still describe the deprecated MVP:

- older ADRs in `02-Decisions/`;
- research-derived product claims;
- most older files in `04-Specs/`;
- `05-Flows/mvp-core-loop.md`;
- `05-Implementation/phase-1-plan.md`;
- the current repository Canvas.

Do not implement behavior from those files merely because they remain in the vault. Reconcile each file against the accepted discussions, then update it, mark it superseded, or remove it from active navigation.

## Repository structure

- `00-Canvas/` — repository Mind Map and its migration ledger
- `01-Closed-Discussions/` — authoritative accepted product, domain, runtime, safety, and validation decisions
- `01-Open-Discussions/` — active discussions; currently includes the migration and implementation reconciliation plan
- `02-Decisions/` — accepted-decision inventory plus ADRs awaiting or carrying explicit reconciliation status
- Repository-wide artifact classifications and migration gates are tracked in [[02-Decisions/repository-artifact-migration-ledger]].
- `03-Research/` — non-authoritative evidence and research notes
- `04-Specs/` — the current AI-native baseline plus specifications at different migration states
- `05-Flows/` — supporting or historical flows; not authoritative unless explicitly reconciled
- `05-Implementation/` — implementation artifacts at different migration states; no current plan is authoritative yet
- `06-References/` — non-authoritative source notes and references

## Change rule

Do not silently change an accepted behavior while migrating documentation or implementing code. A behavior change must reopen or explicitly amend its authoritative source discussion. Migration documents project accepted decisions; they do not create new product semantics.
