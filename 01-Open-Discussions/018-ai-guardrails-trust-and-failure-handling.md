# Discussion 018 — AI Guardrails, Trust, and Failure Handling

## Status
Open for GPT × Claude review.

## Scope
Centralize safety, trust, privacy, reversibility, and failure behavior for Planning AI and Reconcile AI.

## Out of Scope
- detailed prompt design
- provider selection
- final persistence schema
- metric thresholds

## Questions
1. Which actions are always forbidden to AI?
2. Which actions always require explicit confirmation?
3. Which actions may be approved in bulk?
4. Is Drop reversible, and how is history preserved?
5. What happens on timeout, malformed output, refusal, or partial response?
6. What manual fallback exists when AI is unavailable?
7. What is the hard domain allowlist for the pilot?
8. How are medical, legal, financial, crisis, or other high-risk requests handled?
9. What user data may be sent to the model?
10. Are descriptions, notes, Goal titles, and behavior history included by default?
11. What AI inputs/outputs are logged, redacted, or retained?
12. How are prompt injection and hostile Task content handled?
13. How are rule-based facts distinguished from AI inference in UI?
14. How are assumptions, rationale, uncertainty, and reversibility communicated?

## Required Categories
```txt
Allowed without mutation
Requires confirmation
Forbidden
Failure fallback
Privacy boundary
```

## Expected Decisions
- action permission matrix
- domain allowlist
- privacy and retention boundary
- fallback behavior
- reversibility rules
- trust-language requirements

## Dependencies
Should incorporate accepted behavior from Discussions 013, 014, and 017.

## Resolution Template
- Allowed:
- Requires confirmation:
- Forbidden:
- Failure fallback:
- Privacy/retention:
- Mind map changes:
- Specs/ADRs affected:
