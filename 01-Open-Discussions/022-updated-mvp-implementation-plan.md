# Discussion 022 — Updated MVP Implementation Plan

## Status
Open for GPT × Claude review.

## Scope
Convert the resolved AI-native product decisions into a build sequence with bounded vertical slices, team ownership, dependencies, and exit gates.

## Out of Scope
- changing accepted product behavior without reopening its source discussion
- redefining validation thresholds
- introducing new product concepts

## Candidate Sequence
```txt
Foundation
→ Core product domain
→ Manual Task and Routine baseline
→ AI Planning
→ Today execution
→ Basic deterministic Reconcile
→ AI Reconcile
→ Event/analytics completion
→ Pilot readiness
```

## Questions
1. What is the smallest end-to-end vertical slice?
2. Which slice should use mock AI before the real runtime exists?
3. Which existing foundation work remains reusable?
4. What must be removed, amended, or superseded from the old Phase 1 plan?
5. How do frontend, backend, and design work proceed in parallel?
6. What are the milestones and exit gates?
7. Which AI and non-AI fallbacks must exist before pilot launch?
8. When are migrations and API contracts frozen?
9. What tests are mandatory for planning, Routine generation, Today, and Reconcile?
10. What observability must be present before real users?
11. What scope cuts preserve the complete `Plan → Execute → Adapt` loop?
12. Which scope cuts would make the product thesis dishonest?
13. What are the pilot-readiness and release-readiness checklists?

## Required Outputs
- milestone sequence
- ownership by role
- dependency map
- exit gates
- reusable versus superseded work
- test strategy
- pilot-readiness checklist

## Dependencies
This is the final planning discussion and should be resolved only after Discussions 012–021.

## Resolution Template
- Milestones:
- Ownership:
- Exit gates:
- Reused work:
- Superseded work:
- Pilot-readiness gate:
- Mind map changes:
- Specs/ADRs affected:
