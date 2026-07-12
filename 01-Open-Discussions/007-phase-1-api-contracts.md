# Discussion 007 — Phase 1 API Contracts

## Status

Resolved after Claude review and Mahdi approval.

## Result

The accepted contract is formalized in:

- [[04-Specs/api-contracts-phase-1]]

This discussion no longer owns active API decisions.

## Accepted Decisions

### General

- Base path is `/api/v1`.
- JSON uses camelCase.
- Timestamps are UTC ISO-8601 instants.
- Local dates use `YYYY-MM-DD`.
- Success responses return direct resource bodies without generic envelopes.
- Errors use RFC 9457-style Problem Details with a stable `code` and `traceId`.
- Validation failures consistently use HTTP 400, not 422.
- Non-owned resources return 404 without revealing whether they exist.
- Phase 1 uses neither idempotency keys nor optimistic-locking version fields.
- Mutations use database/state guards such as `status = ACTIVE`; harmless conflicts may return 409.

### Authentication and current user

- `POST /auth/otp/request` returns `202 Accepted` with backend-owned `resendAvailableAt`.
- OTP verification returns the current User representation and writes the JWT cookie.
- Do not return `isNewUser` or frontend route names.
- Frontend routing is derived only from `User.onboardingStatus`.
- `GET /users/me` is the single current-user endpoint.
- `/auth/me` does not exist.

### Goals and tasks

- Goal and Task lists are unpaginated in Phase 1.
- Task representations embed a nullable minimal Goal summary containing only `id` and `title`.
- Task updates use one `PATCH /tasks/{taskId}` endpoint.
- Event mapping for `plannedForDate` changes is explicit:

```txt
null → date                 TASK_PLANNED
date → different date      TASK_PLANNED
date → null                 TASK_UNSCHEDULED
title-only change           no behavioral event
```

- `TASK_ALREADY_RESOLVED` is removed; use `TASK_NOT_ACTIVE`.
- `GOAL_NOT_OWNED` is removed; use `RESOURCE_NOT_FOUND`.

### Drop and Undo

- Undo is frontend-only.
- The frontend delays the actual DROP request for the toast window.
- If Undo is pressed, no backend request and no event occur.
- If the tab closes during the window, the task remains ACTIVE and may reappear later.
- Completed or dropped tasks remain non-reopenable.

### Reconcile

- Opening Reconcile is `POST /reconcile/open`, because opening records `RECONCILE_SHOWN`.
- Done, Carry, and Drop use one command endpoint:

```txt
POST /reconcile/tasks/{taskId}/resolve
```

- `RECONCILE_STARTED` remains, but no separate start endpoint exists.
- The backend appends `RECONCILE_STARTED` idempotently on the first successful resolve of that local day.
- Resolve responses include `remainingCount`.
- Before appending `RECONCILE_COMPLETED`, the backend performs a fresh eligibility query against current database state in the same transaction.
- The backend never trusts a client-held snapshot count for the terminal decision.
- If eligible tasks remain, completion is not recorded and the updated count or task set is returned.

### App opened event

- `POST /app/open` is a narrow event endpoint, not a bundled bootstrap response.
- `APP_OPENED` is recorded at most once per user local calendar day in Phase 1.
- This event represents daily presence for retention analysis, not actual session counting.

### Scope

- Weekly Review / Reset has no Phase 1 endpoint.
- Its absence is intentional, not an omitted contract.
- AI planning, calendar integration, refresh-token endpoints, offline mutation, and Phase 2 behavior remain excluded.

### Validation limits

```txt
Goal title: 120 characters
Task title: 240 characters
Display name: 80 characters
```

## Superseded Options

The following proposals are rejected:

- backend-persisted Drop followed by reopen/undo
- GET endpoints with Reconcile side effects
- explicit `/reconcile/start`
- client-triggered `/reconcile/complete`
- trusting the original Reconcile snapshot when deciding completion
- `/auth/me` alongside `/users/me`
- `isNewUser` in auth responses
- generic BFF-style bootstrap response
- Phase 1 pagination
- idempotency-key infrastructure
- optimistic-locking version fields
- 422 validation responses
- redundant ownership-specific error codes

## Next Step

Use [[04-Specs/api-contracts-phase-1]] as the API source of truth while creating the Phase 1 implementation plan.