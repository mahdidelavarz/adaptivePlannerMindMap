# Discussion 013 — AI Planning Entry and Conversation Flow

## Status
Open for GPT × Claude review.

## Scope
Define how a user enters the AI-assisted planning flow and moves from an unclear intention to a reviewable draft.

## Out of Scope
- final AI JSON contract
- Task/Routine execution behavior
- Reconcile
- database and API implementation

## Questions
1. Does the user start with a Goal, a free-form intention, or both?
2. What is the first prompt and empty-state message?
3. Which clarifying questions are mandatory, optional, or forbidden?
4. What is the maximum question count before generating a draft?
5. Is the interaction conversational, form-based, or hybrid?
6. Can the user directly request only a Task or only a Routine?
7. Where is manual creation available?
8. What is the exact pilot domain allowlist?
9. What happens for an out-of-scope domain?
10. What happens when user input is vague, contradictory, or incomplete?
11. Can the user skip clarification and request a draft immediately?
12. How does the user exit, restart, or recover from a failed flow?

## Expected Decisions
- entry points
- conversation states
- clarification limits
- domain boundaries
- manual fallback
- failure and exit paths

## Dependencies
Requires the definitions accepted in [[01-Open-Discussions/012-core-product-model]].

## Resolution Template
- Accepted entry model:
- Clarification policy:
- Domain allowlist:
- Fallback behavior:
- Mind map changes:
- Specs/ADRs affected:
