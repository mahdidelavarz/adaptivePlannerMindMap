# Ownership Matrix

## Status

`WORKSTREAM_I_APPROVED — ROLE LABELS REQUIRE NAMED ASSIGNEES BEFORE MILESTONE ENTRY`

Role labels must be replaced with named individuals before the applicable milestone begins. Mahdi is currently the repository/Product owner and recorded cross-role documentation approver; implementation accountability still requires explicit assignees.

| Area | Accountable | Responsible contributors | Mandatory reviewers | Approval point |
|---|---|---|---|---|
| product scope and traceability | Product owner | Product/Frontend | Design, Backend | every slice lock |
| interaction flows and accessibility | Product Design | Design + Frontend | Product, Safety where applicable | design handoff/exit |
| frontend contracts/state machines | Frontend owner | Frontend | Backend, Design, Security | slice integration |
| domain/persistence/transactions | Backend owner | Backend | Product, Security | migration and slice exit |
| authentication/authorization | Backend owner | Backend + Frontend | Security/Privacy | M1 and production gate |
| events/observability/retention | Backend owner | Backend + Research | Security/Privacy, Research | schema lock/M8 |
| AI runtime/reliability/cost | Backend owner | Backend + Frontend | Safety, Security/Privacy, Product | M5/M7 exits |
| AI UX and manual escape | Product owner | Design + Frontend + Backend | Safety | M4/M5/M7 exits |
| crisis/high-risk boundary | Safety/Policy owner | Safety + Product + Backend | Security/Privacy, Research | any real-user AI exposure/M9 |
| metrics and pilot analysis | Pilot Research owner | Research + Backend/Product | Safety, Security/Privacy | M8/M9 |
| operations/support/incident/rollback | Backend owner | Backend + Product | Safety, Security/Privacy | M8/M9 |

## Milestone accountability

| Milestone | Accountable role | Required exit approvers |
|---|---|---|
| M1 | Backend owner | Product, Frontend, Security/Privacy |
| M2 | Product/Frontend owner | Backend, Design, Security/Privacy |
| M3 | Backend owner | Product, Frontend, Design |
| M4 | Product/Frontend owner | Backend, Design, Safety |
| M5 | Backend owner | Product, Frontend, Safety, Security/Privacy |
| M6 | Backend owner | Product, Frontend, Design, Safety, Research |
| M7 | Backend owner | Product, Frontend, Design, Safety, Security/Privacy, Research |
| M8 | Pilot Research owner | Product, Backend, Safety, Security/Privacy |
| M9 | Product owner | all accountable owners and mandatory reviewers |

## Handoff rule

Each handoff records artifact/version, assumptions, unresolved items, test evidence, consumer acknowledgement, and rollback owner. “Everyone owns it” is not a valid accountable assignment.
