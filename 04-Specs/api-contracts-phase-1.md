# Phase 1 API Contracts

## Status

Accepted for Phase 1.

Formalized from [[01-Open-Discussions/007-phase-1-api-contracts]] after Claude review and Mahdi approval.

## Purpose

Define the REST contract between the React client and Spring Boot backend for the accepted Phase 1 product surface.

Use with:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/auth-phase-1]]
- [[04-Specs/phase-1-implementation-stack]]

This spec owns endpoint boundaries, request/response shapes, status codes, error contracts, mutation semantics, and API-level concurrency rules.

---

# 1. Global Conventions

## Base path

```txt
/api/v1
```

## Transport

- JSON over HTTPS
- camelCase JSON fields
- UTC ISO-8601 timestamps
- `YYYY-MM-DD` local dates
- authenticated requests use the accepted HttpOnly JWT cookie
- unsafe methods use the accepted same-origin and Origin/Referer controls

## Ownership

The authenticated user is derived from the JWT.

Never accept a client-supplied `userId` for user-owned resources.

A resource owned by another user returns:

```txt
404 RESOURCE_NOT_FOUND
```

Do not reveal that the resource exists.

## Success bodies

Return direct resource bodies rather than generic `{ success, data }` envelopes.

Use `204 No Content` when no response body is needed.

## Error contract

Use RFC 9457-style Problem Details:

```json
{
  "type": "https://adaptiveplanner/errors/validation",
  "title": "Request validation failed",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "traceId": "01J...",
  "errors": {
    "title": ["must not be blank"]
  }
}
```

Required fields:

```txt
type
title
status
code
traceId
```

Optional fields:

```txt
detail
instance
errors
retryAfterSeconds
```

Accepted stable codes:

```txt
VALIDATION_ERROR
UNAUTHENTICATED
FORBIDDEN
RESOURCE_NOT_FOUND
CONFLICT
RATE_LIMITED
OTP_INVALID
OTP_EXPIRED
OTP_ALREADY_USED
OTP_ATTEMPTS_EXCEEDED
OTP_SEND_FAILED
SESSION_EXPIRED
SESSION_REVOKED
TASK_NOT_ACTIVE
RECONCILE_NOT_AVAILABLE
RECONCILE_ALREADY_SHOWN_TODAY
INTERNAL_ERROR
```

Validation failures use HTTP 400 consistently. Phase 1 does not use 422.

---

# 2. Authentication

Authentication behavior is governed by [[04-Specs/auth-phase-1]].

## Request OTP

```txt
POST /api/v1/auth/otp/request
```

Request:

```json
{
  "phoneNumber": "09121234567",
  "purpose": "LOGIN"
}
```

Success:

```txt
202 Accepted
```

```json
{
  "accepted": true,
  "resendAvailableAt": "2026-07-12T10:05:00Z"
}
```

Rules:

- response does not reveal whether the user exists
- cooldown truth comes from the backend
- provider identifiers and challenge internals are never returned

## Verify OTP

```txt
POST /api/v1/auth/otp/verify
```

Request:

```json
{
  "phoneNumber": "09121234567",
  "code": "123456",
  "purpose": "LOGIN"
}
```

Success:

```txt
200 OK
```

The backend writes the JWT cookie and returns:

```json
{
  "user": {
    "id": "018f...",
    "displayName": null,
    "timezone": "Asia/Tehran",
    "onboardingStatus": "NOT_STARTED",
    "onboardingCompletedAt": null,
    "createdAt": "2026-07-12T09:00:00Z"
  }
}
```

Do not return:

```txt
isNewUser
nextRoute
JWT value
sessionEpoch
normalizedPhone
```

The frontend routes only from `onboardingStatus`.

## Logout current browser

```txt
POST /api/v1/auth/logout
```

```txt
204 No Content
```

Behavior:

- clear JWT cookie
- do not increment `sessionEpoch`
- remain idempotent

## Logout all / forced revoke

```txt
POST /api/v1/auth/logout-all
```

```txt
204 No Content
```

Behavior:

- increment `User.sessionEpoch`
- clear current cookie

The endpoint may be implemented for operational support even if the Phase 1 UI does not expose it.

There is no `/auth/me`, refresh endpoint, password endpoint, email-auth endpoint, or OAuth endpoint.

---

# 3. Current User and Onboarding

## Get current user

```txt
GET /api/v1/users/me
```

```json
{
  "id": "018f...",
  "displayName": "Mahdi",
  "timezone": "Asia/Tehran",
  "onboardingStatus": "IN_PROGRESS",
  "onboardingCompletedAt": null,
  "createdAt": "2026-07-12T09:00:00Z"
}
```

## Update current user

```txt
PATCH /api/v1/users/me
```

Request:

```json
{
  "displayName": "Mahdi",
  "timezone": "Asia/Tehran"
}
```

Rules:

- omitted fields remain unchanged
- explicit null is allowed only for nullable fields
- `normalizedPhone` cannot be changed here
- timezone must be a valid IANA identifier
- `displayName` maximum is 80 characters

Success returns the updated User representation.

## Start onboarding

```txt
POST /api/v1/onboarding/start
```

Success:

```json
{
  "onboardingStatus": "IN_PROGRESS"
}
```

Rules:

- append `ONBOARDING_STARTED` atomically with the status mutation
- remain idempotent
- do not rely on route viewing as a mutation

## Complete onboarding

```txt
POST /api/v1/onboarding/complete
```

Request:

```json
{}
```

Success:

```json
{
  "onboardingStatus": "COMPLETED",
  "onboardingCompletedAt": "2026-07-12T09:15:00Z"
}
```

Rules:

- works even when no goal or task was created
- status mutation and `ONBOARDING_COMPLETED` append are atomic
- repeated calls do not duplicate the event

---

# 4. Goal Contracts

## Goal representation

```json
{
  "id": "018f...",
  "title": "Become a stronger engineer",
  "source": "MANUAL",
  "createdAt": "2026-07-12T09:20:00Z",
  "updatedAt": "2026-07-12T09:20:00Z"
}
```

The client cannot create `AI_GENERATED` goals in Phase 1.

## Create goal

```txt
POST /api/v1/goals
```

Request:

```json
{
  "title": "Become a stronger engineer"
}
```

Success:

```txt
201 Created
Location: /api/v1/goals/{goalId}
```

Returns the created Goal.

## List goals

```txt
GET /api/v1/goals
```

```json
{
  "items": [
    {
      "id": "018f...",
      "title": "Become a stronger engineer",
      "source": "MANUAL",
      "createdAt": "2026-07-12T09:20:00Z",
      "updatedAt": "2026-07-12T09:20:00Z"
    }
  ]
}
```

No pagination in Phase 1.

## Get goal

```txt
GET /api/v1/goals/{goalId}
```

Returns the Goal or 404.

## Update goal

```txt
PATCH /api/v1/goals/{goalId}
```

Request:

```json
{
  "title": "Become a stronger backend engineer"
}
```

Goal title maximum is 120 characters.

## Delete goal

```txt
DELETE /api/v1/goals/{goalId}
```

```txt
204 No Content
```

Phase 1 rule:

- deleting a Goal must not delete Tasks
- affected Tasks become standalone by setting `goalId = null`
- historical behavioral events remain untouched
- the operation must be transactional

---

# 5. Task Contracts

## Task representation

```json
{
  "id": "018f...",
  "title": "Read chapter 3",
  "status": "ACTIVE",
  "plannedForDate": "2026-07-12",
  "carryCount": 0,
  "source": "MANUAL",
  "goal": {
    "id": "018f-goal...",
    "title": "Become a stronger engineer"
  },
  "completedAt": null,
  "droppedAt": null,
  "createdAt": "2026-07-12T09:30:00Z",
  "updatedAt": "2026-07-12T09:30:00Z"
}
```

`goal` is nullable and contains only `id` and `title`.

## Create task

```txt
POST /api/v1/tasks
```

Request:

```json
{
  "title": "Read chapter 3",
  "goalId": null,
  "plannedForDate": "2026-07-12"
}
```

Success:

```txt
201 Created
Location: /api/v1/tasks/{taskId}
```

Rules:

- title maximum is 240 characters
- `goalId` is optional
- `plannedForDate` is optional
- referenced Goal must belong to the authenticated user
- create Task and append `TASK_CREATED` atomically
- if initially planned, metadata or event behavior must follow the accepted implementation rule without duplicating semantic events

## List tasks

```txt
GET /api/v1/tasks
```

Supported filters may include:

```txt
status
goalId
plannedForDate
```

Response:

```json
{
  "items": []
}
```

No pagination in Phase 1.

## Get task

```txt
GET /api/v1/tasks/{taskId}
```

Returns the Task or 404.

## Update task

```txt
PATCH /api/v1/tasks/{taskId}
```

Example:

```json
{
  "title": "Read chapter 4",
  "goalId": "018f-goal...",
  "plannedForDate": "2026-07-13"
}
```

Only ACTIVE tasks may be edited.

Explicit event mapping:

| Previous `plannedForDate` | New value | Event |
|---|---|---|
| null | date | `TASK_PLANNED` |
| date | different date | `TASK_PLANNED` |
| date | null | `TASK_UNSCHEDULED` |
| unchanged | unchanged | none |
| title-only change | unchanged | none |

Relevant state mutation and event append must be atomic.

## Complete task outside Reconcile

```txt
POST /api/v1/tasks/{taskId}/complete
```

Success returns the completed Task.

Rules:

- only ACTIVE tasks can complete
- mutation and `TASK_COMPLETED` append are atomic
- repeated/conflicting attempts return the accepted Task-not-active conflict

## Delete task

Generic task deletion is excluded from Phase 1 unless separately approved. Drop is a Reconcile outcome, not ordinary deletion.

---

# 6. Today

## Get Today

```txt
GET /api/v1/today
```

Response:

```json
{
  "localDate": "2026-07-12",
  "timezone": "Asia/Tehran",
  "tasks": []
}
```

Rules:

- use the user's local date
- return ACTIVE tasks where `plannedForDate` equals that date
- empty Today is a valid response

---

# 7. Reconcile

## Open Reconcile

```txt
POST /api/v1/reconcile/open
```

This is a command because it may append `RECONCILE_SHOWN`.

Possible success:

```json
{
  "available": true,
  "localDate": "2026-07-12",
  "tasks": [
    {
      "id": "018f...",
      "title": "Read chapter 3",
      "status": "ACTIVE",
      "plannedForDate": "2026-07-11",
      "carryCount": 1,
      "goal": null
    }
  ],
  "remainingCount": 1
}
```

When unavailable:

```json
{
  "available": false,
  "localDate": "2026-07-12",
  "tasks": [],
  "remainingCount": 0
}
```

Rules:

- eligibility uses current DB state and the user's local date
- append `RECONCILE_SHOWN` at most once per local day
- after Reconcile has been shown that day, do not show it again that day
- no persistent ReconcileSession is created

## Skip Reconcile

```txt
POST /api/v1/reconcile/skip
```

Request:

```json
{}
```

Success:

```txt
204 No Content
```

Rules:

- append `RECONCILE_SKIPPED` idempotently for the local day
- no same-day retrigger
- unresolved tasks remain ACTIVE

## Resolve a Reconcile task

```txt
POST /api/v1/reconcile/tasks/{taskId}/resolve
```

### Done

```json
{
  "decision": "DONE"
}
```

### Carry

```json
{
  "decision": "CARRY",
  "plannedForDate": "2026-07-12"
}
```

### Drop

```json
{
  "decision": "DROP"
}
```

Success:

```json
{
  "task": {
    "id": "018f...",
    "status": "COMPLETED"
  },
  "remainingCount": 0,
  "reconcileCompleted": true
}
```

Rules:

- only an eligible ACTIVE task owned by the user can resolve
- all three decisions use this one command endpoint
- mutation and Task event append are atomic
- on the first successful resolve of the user's local day, append `RECONCILE_STARTED` idempotently
- no separate start endpoint exists
- before appending `RECONCILE_COMPLETED`, perform a fresh eligibility query against current database state in the same transaction
- never trust the original client snapshot or client count for completion
- if any eligible task remains, do not append completion
- response returns the fresh `remainingCount`
- `RECONCILE_COMPLETED` is appended once when the fresh count reaches zero

## Drop Undo rule

Undo is not a backend operation.

Frontend behavior:

1. user presses Drop
2. hold the DROP request in local state for the toast window, approximately 4–6 seconds
3. if Undo is pressed, cancel the pending request
4. if the window expires, send the DROP resolve command

If the tab closes during the pending window, do not flush with `sendBeacon`; the Task stays ACTIVE.

Therefore an undone Task was never persisted as DROPPED and the no-reopen invariant remains intact.

There is no undo or reopen endpoint.

---

# 8. App Presence

## Record app open

```txt
POST /api/v1/app/open
```

```txt
204 No Content
```

Rules:

- append `APP_OPENED` at most once per user local calendar day
- reuse the event-log local-date lookup pattern
- repeated calls that day remain idempotent
- this represents daily presence for retention analysis, not true session counting
- do not bundle user, Today, Reconcile, or bootstrap data into this endpoint

---

# 9. Concurrency and Idempotency

Phase 1 does not add:

```txt
idempotency keys
optimistic-locking version fields
client-generated mutation ids
```

Use transaction and state guards, for example:

```sql
UPDATE task
SET status = 'COMPLETED', completed_at = :now
WHERE id = :taskId
  AND user_id = :userId
  AND status = 'ACTIVE';
```

Rules:

- a zero-row guarded mutation returns a conflict such as `TASK_NOT_ACTIVE`
- duplicate retries must never create duplicate events or corrupt state
- OTP-specific concurrency follows the auth spec
- Reconcile completion always rechecks eligibility in current DB state

---

# 10. Explicitly Excluded Endpoints

Phase 1 has no endpoints for:

```txt
Weekly Review / Reset
AI planning
AI suggestions
calendar integration
routines
habits
time blocking
deadlines
notifications
refresh tokens
OAuth
email login
password login
offline mutation sync
Reconcile sessions
reopening terminal tasks
backend Drop undo
```

Weekly Review's absence is intentional. It is not a forgotten contract.

---

# 11. Endpoint Inventory

```txt
POST   /api/v1/auth/otp/request
POST   /api/v1/auth/otp/verify
POST   /api/v1/auth/logout
POST   /api/v1/auth/logout-all

GET    /api/v1/users/me
PATCH  /api/v1/users/me

POST   /api/v1/onboarding/start
POST   /api/v1/onboarding/complete

POST   /api/v1/goals
GET    /api/v1/goals
GET    /api/v1/goals/{goalId}
PATCH  /api/v1/goals/{goalId}
DELETE /api/v1/goals/{goalId}

POST   /api/v1/tasks
GET    /api/v1/tasks
GET    /api/v1/tasks/{taskId}
PATCH  /api/v1/tasks/{taskId}
POST   /api/v1/tasks/{taskId}/complete

GET    /api/v1/today

POST   /api/v1/reconcile/open
POST   /api/v1/reconcile/skip
POST   /api/v1/reconcile/tasks/{taskId}/resolve

POST   /api/v1/app/open
```

---

# 12. Required Contract Tests

At minimum:

```txt
OTP request returns resendAvailableAt
OTP verify returns user without isNewUser or nextRoute
/users/me is canonical and /auth/me is absent
non-owned resources return 404
validation failures use 400 Problem Details
Task embeds only minimal nullable Goal summary
Task PATCH produces exact planned/unscheduled event mapping
Task title-only PATCH produces no behavioral event
Drop Undo sends no backend mutation when cancelled
Reconcile open records shown only once per local day
first successful resolve records started once
resolve uses fresh remaining eligibility
newly eligible straggler prevents premature reconcile completion
completion event is emitted once
APP_OPENED is recorded at most once per local day
state-guarded duplicate mutation cannot corrupt Task or duplicate its event
```

## Next Step

Use this contract while creating the Phase 1 implementation plan and backend/frontend scaffold.