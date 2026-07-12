# Discussion 007 — Phase 1 API Contracts

## Status

Waiting for Claude review.

## Context

The following Phase 1 decisions are already formalized:

- [[04-Specs/day-0-onboarding]]
- [[04-Specs/reconcile-ux]]
- [[04-Specs/data-model-phase-1]]
- [[04-Specs/auth-phase-1]]
- [[04-Specs/phase-1-implementation-stack]]

This discussion defines the proposed REST contract between the React client and Spring Boot backend.

It does not reopen:

- the product scope
- the data model
- OTP-only authentication
- JWT cookie transport
- the Reconcile behavior
- the technology stack

The purpose is to remove ambiguity before frontend and backend implementation begin.

---

# Locked API Principles

## Base path

```txt
/api/v1
```

All Phase 1 application endpoints use the versioned prefix.

## Transport

```txt
JSON over HTTPS
```

Rules:

- request and response bodies use JSON unless the endpoint intentionally returns no body
- timestamps use UTC ISO-8601 instants
- local dates use `YYYY-MM-DD`
- authenticated requests use the HttpOnly JWT cookie
- the frontend never receives or stores the JWT value
- unsafe methods require the accepted same-origin / Origin-Referer security checks

## Naming

JSON uses camelCase.

Examples:

```txt
plannedForDate
onboardingStatus
carryCount
```

## Resource ownership

The authenticated user is inferred from the JWT.

Do not accept `userId` from the frontend for user-owned resource mutations.

Bad:

```json
{
  "userId": "...",
  "title": "..."
}
```

Good:

```json
{
  "title": "..."
}
```

The backend always scopes goals, tasks, Today, and Reconcile operations to the authenticated user.

---

# Response Conventions

## Success responses

Use direct resource bodies rather than wrapping every response in a generic envelope.

Preferred:

```json
{
  "id": "...",
  "title": "Read chapter 3"
}
```

Avoid:

```json
{
  "success": true,
  "data": {
    "id": "..."
  }
}
```

A generic success envelope adds ceremony without useful information.

## Empty successful responses

Use:

```txt
204 No Content
```

for operations such as logout where no response body is required.

## Error responses

Use RFC 9457-style Problem Details.

Proposed shape:

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

`code` is the stable machine-readable frontend contract.

The frontend must not branch on human-readable `title` or `detail` text.

## Proposed error codes

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
TASK_ALREADY_RESOLVED
RECONCILE_NOT_AVAILABLE
RECONCILE_ALREADY_SHOWN_TODAY
GOAL_NOT_OWNED
INTERNAL_ERROR
```

Claude should review which codes are redundant or too implementation-specific.

---

# Authentication Contracts

The behavior is governed by [[04-Specs/auth-phase-1]].

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

Proposed success response:

```txt
202 Accepted
```

```json
{
  "accepted": true,
  "resendAvailableAt": "2026-07-12T10:05:00Z"
}
```

Notes:

- the response does not reveal whether the user exists
- `resendAvailableAt` lets the frontend render cooldown from backend truth
- the response does not return challenge internals or provider identifiers

Potential alternative:

```txt
204 No Content
```

GPT preference: keep the small response with `resendAvailableAt`; otherwise the frontend must guess cooldown state.

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

Proposed success:

```txt
200 OK
```

The backend writes the JWT cookie and returns the current user state:

```json
{
  "user": {
    "id": "018f...",
    "displayName": null,
    "timezone": "Asia/Tehran",
    "onboardingStatus": "NOT_STARTED"
  },
  "isNewUser": true,
  "nextRoute": "/onboarding"
}
```

Open question:

Should `nextRoute` be returned by the backend, or should the frontend derive navigation from `onboardingStatus`?

GPT preference: do not return `nextRoute`; return domain state and let the frontend route. `nextRoute` couples the API to frontend paths.

Preferred final body:

```json
{
  "user": {
    "id": "018f...",
    "displayName": null,
    "timezone": "Asia/Tehran",
    "onboardingStatus": "NOT_STARTED"
  },
  "isNewUser": true
}
```

## Current session

```txt
GET /api/v1/auth/me
```

Success:

```json
{
  "id": "018f...",
  "displayName": "Mahdi",
  "timezone": "Asia/Tehran",
  "onboardingStatus": "COMPLETED"
}
```

This endpoint supports:

- application bootstrap
- reload with an existing cookie
- session validation
- current user state

Open question:

Should this be `/auth/me` or `/users/me`?

GPT preference: use `/users/me` as the canonical current-user resource and avoid duplicate endpoints. Auth owns login/logout; User owns profile state.

## Logout current browser

```txt
POST /api/v1/auth/logout
```

Response:

```txt
204 No Content
```

Behavior:

- clear the JWT cookie
- do not increment `sessionEpoch`

The operation should remain idempotent. Logging out while already logged out may still return 204.

## Logout all / force revoke

Proposed endpoint:

```txt
POST /api/v1/auth/logout-all
```

Response:

```txt
204 No Content
```

Behavior:

- increment `User.sessionEpoch`
- clear the current cookie

Open question:

The auth spec allows Mahdi to decide whether logout-all appears in Phase 1 UI. Should the endpoint still exist even if the UI does not expose it?

GPT preference: implement it only if required operationally for forced revocation or tester support. Do not expose unused public API merely because it might be useful someday.

---

# Current User and Onboarding Contracts

## Get current user

```txt
GET /api/v1/users/me
```

Response:

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

Do not expose:

- normalized phone number by default
- `sessionEpoch`
- internal auth fields

Open question:

Should the user be allowed to view their phone number in settings?

GPT preference: it may be returned only from a dedicated account/settings representation if the UI needs it, masked where appropriate. It does not belong in every bootstrap response.

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
- explicit `null` is permitted only where the field is nullable
- `normalizedPhone` cannot be changed through this endpoint
- timezone must be a valid IANA identifier

Response:

```txt
200 OK
```

with the updated user representation.

## Start onboarding

Potential endpoint:

```txt
POST /api/v1/onboarding/start
```

Response:

```json
{
  "onboardingStatus": "IN_PROGRESS",
  "startedAt": "2026-07-12T09:05:00Z"
}
```

Open question:

Does Phase 1 need an explicit start endpoint, or should onboarding become `IN_PROGRESS` automatically when the authenticated user first loads the onboarding route?

GPT preference: avoid route-view mutations. Use an explicit endpoint if `ONBOARDING_STARTED` is an accepted event and must be reliable.

## Complete onboarding

```txt
POST /api/v1/onboarding/complete
```

Request:

```json
{}
```

Response:

```json
{
  "onboardingStatus": "COMPLETED",
  "onboardingCompletedAt": "2026-07-12T09:15:00Z"
}
```

The endpoint must work even if the user created no goal and no task, because every Day-0 creation step is optional.

This operation is idempotent. Repeating it should not duplicate `ONBOARDING_COMPLETED` events.

---

# Goal Contracts

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

`AI_GENERATED` may exist in the schema but clients cannot create AI-generated goals in Phase 1.

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

Response:

```txt
201 Created
Location: /api/v1/goals/{goalId}
```

with the created Goal body.

## List goals

```txt
GET /api/v1/goals
```

Proposed response:

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

Open question:

Does this small list need pagination in Phase 1?

GPT preference: no pagination initially. The product does not encourage hundreds of goals, and pagination adds API/UI ceremony without evidence.

## Get goal

```txt
GET /api/v1/goals/{goalId}
```

Response:

```txt
200 OK
```

or `404 RESOURCE_NOT_FOUND` when the goal does not exist or is not owned by the current user.

Do not reveal whether another user's goal exists.

## Update goal

```txt
PATCH /api/v1/goals/{goalId}
```

Request:

```json
{
  "title": "Become a strong software engineer"
}
```

Response: updated Goal.

## Delete goal

The data model intentionally left goal deletion behavior for implementation.

Options:

### Option A — exclude goal deletion from Phase 1

No endpoint.

### Option B — delete goal but keep tasks

```txt
DELETE /api/v1/goals/{goalId}
```

Behavior:

- set `Task.goalId = null` for active and historical tasks
- keep events
- delete goal

### Option C — reject deletion while tasks reference the goal

Return `409 CONFLICT` until tasks are detached.

GPT preference: exclude goal deletion from Phase 1 unless the accepted UI requires it. Creating lifecycle complexity for goals before users prove they care about goals would be impressively on-brand for software projects, but still unnecessary.

---

# Task Contracts

## Task representation

```json
{
  "id": "018f...",
  "goal": {
    "id": "018a...",
    "title": "Become a stronger engineer"
  },
  "title": "Read chapter 3",
  "status": "ACTIVE",
  "plannedForDate": "2026-07-12",
  "carryCount": 0,
  "source": "MANUAL",
  "completedAt": null,
  "droppedAt": null,
  "createdAt": "2026-07-12T09:30:00Z",
  "updatedAt": "2026-07-12T09:30:00Z"
}
```

Open question:

Should task responses embed a small Goal summary as above, or return only `goalId`?

GPT preference: embed a minimal nullable Goal summary for list/Today/Reconcile screens. This prevents the frontend from joining goals repeatedly while avoiding full nested resources.

## Create task

```txt
POST /api/v1/tasks
```

Request:

```json
{
  "title": "Read chapter 3",
  "goalId": "018a...",
  "plannedForDate": "2026-07-12"
}
```

Both optional associations may be omitted:

```json
{
  "title": "Read chapter 3"
}
```

Response:

```txt
201 Created
Location: /api/v1/tasks/{taskId}
```

with the created Task.

Rules:

- `goalId` must belong to the current user
- `plannedForDate` is date-only
- the backend sets `source = MANUAL`
- client cannot provide status, carryCount, completedAt, droppedAt, userId, or source

## List tasks

```txt
GET /api/v1/tasks
```

Proposed query parameters:

```txt
status=ACTIVE|COMPLETED|DROPPED
goalId={uuid}
plannedForDate=YYYY-MM-DD
scheduled=true|false
```

Proposed response:

```json
{
  "items": []
}
```

Open questions:

1. Should task listing support multiple statuses?
2. Is pagination required?
3. Should completed/dropped tasks be exposed in Phase 1 UI at all?

GPT preference:

- support one optional status filter
- no pagination during the 14-day validation
- keep completed/dropped queryable for debugging and future history UI, even if the initial UI does not show a full archive

## Get task

```txt
GET /api/v1/tasks/{taskId}
```

Return 404 for missing or non-owned tasks.

## Edit active task

```txt
PATCH /api/v1/tasks/{taskId}
```

Request may include:

```json
{
  "title": "Read chapters 3 and 4",
  "goalId": null,
  "plannedForDate": "2026-07-13"
}
```

Rules:

- only ACTIVE tasks can be edited
- omitted fields remain unchanged
- explicit `goalId: null` detaches the goal
- explicit `plannedForDate: null` unschedules the task
- title must not be editable after completion/drop
- status is not editable through generic PATCH

Response: updated Task.

Open question:

Should changing `plannedForDate` through generic PATCH append `TASK_PLANNED` / `TASK_UNSCHEDULED`, or should planning have dedicated action endpoints?

GPT preference: service-layer behavior should inspect the change and append the correct event atomically. Separate endpoints are clearer for actions but would make simple editing unnecessarily fragmented.

## Complete task outside Reconcile

```txt
POST /api/v1/tasks/{taskId}/complete
```

Response: updated Task with `status = COMPLETED`.

Rules:

- only ACTIVE tasks may be completed
- operation is atomic with `TASK_COMPLETED`
- repeated completion should return either the existing completed task or a conflict

Open question:

Should terminal action endpoints be idempotent?

GPT preference: make them idempotent where practical. Double-taps and network retries should not turn ordinary user behavior into a 409 festival. Returning the already-completed representation is acceptable if the requested terminal state already matches.

## Drop task outside Reconcile

Potential endpoint:

```txt
POST /api/v1/tasks/{taskId}/drop
```

Open question:

Does Phase 1 allow dropping only inside Reconcile, or from normal task management too?

GPT preference: the accepted UX centers Drop inside Reconcile. Do not expose a general drop action unless the UI explicitly needs it.

## Undo drop

The Reconcile UX includes Drop + Undo.

Potential endpoint:

```txt
POST /api/v1/tasks/{taskId}/undo-drop
```

Behavior:

- only recently dropped task may be restored
- status returns to ACTIVE
- droppedAt becomes null
- no generic reopen behavior exists

This conflicts with the accepted data-model rule that dropped tasks cannot be reopened in Phase 1.

Therefore one of these must be made explicit:

### Option A — Undo is client-delayed commit

- UI shows “Dropped. Undo”
- backend drop request is delayed briefly
- Undo cancels before the mutation is sent

### Option B — bounded server-side undo

- Drop is persisted immediately
- a narrow undo endpoint is permitted for a short window
- this is not considered generic reopening
- corresponding event behavior must be defined

### Option C — optimistic UI with compensating mutation

- backend persists Drop
- Undo sends a specific reversal action
- append a reversal event

GPT preference: Option A is simplest only if navigation/network behavior is carefully handled. Option B is more reliable but requires changing the formal data-model rule and defining a reversal event. Claude should review this carefully because the current UX and data model are not fully aligned here.

---

# Today Contract

## Get Today view

```txt
GET /api/v1/today
```

The backend determines the user's current local date from `User.timezone`.

Proposed response:

```json
{
  "localDate": "2026-07-12",
  "timezone": "Asia/Tehran",
  "tasks": [
    {
      "id": "018f...",
      "goal": null,
      "title": "Read chapter 3",
      "status": "ACTIVE",
      "plannedForDate": "2026-07-12",
      "carryCount": 0
    }
  ]
}
```

Rules:

- only ACTIVE tasks planned for the current local date are returned
- unresolved older tasks are not silently mixed into Today before Reconcile
- empty Today is a valid 200 response with `tasks: []`

Open question:

Should Today return Reconcile trigger state in the same response?

Example:

```json
{
  "localDate": "2026-07-12",
  "tasks": [],
  "reconcile": {
    "required": true,
    "unresolvedTaskCount": 3
  }
}
```

GPT preference: a bootstrap/dashboard endpoint may reduce requests, but coupling Today and Reconcile risks turning one endpoint into an application-state junk drawer. Prefer a dedicated Reconcile availability endpoint unless frontend startup latency proves meaningful.

---

# Reconcile Contracts

Reconcile follows [[04-Specs/reconcile-ux]] and does not use a persistent ReconcileSession model.

## Get Reconcile availability

```txt
GET /api/v1/reconcile
```

Proposed response when available:

```json
{
  "available": true,
  "localDate": "2026-07-12",
  "timezone": "Asia/Tehran",
  "unresolvedTaskCount": 3,
  "tasks": [
    {
      "id": "018f...",
      "goal": null,
      "title": "Read chapter 3",
      "plannedForDate": "2026-07-11",
      "carryCount": 1
    }
  ]
}
```

Response when not available:

```json
{
  "available": false,
  "reason": "NO_UNRESOLVED_TASKS"
}
```

Possible reasons:

```txt
NO_UNRESOLVED_TASKS
ALREADY_SHOWN_TODAY
```

Open question:

Does a GET endpoint that returns available Reconcile tasks also append `RECONCILE_SHOWN`?

A GET request should normally be safe and side-effect free, but the product requires reliable shown/no-retrigger behavior.

Options:

### Option A — GET is read-only, frontend explicitly marks shown

```txt
GET /api/v1/reconcile
POST /api/v1/reconcile/shown
```

### Option B — use a POST command to open Reconcile

```txt
POST /api/v1/reconcile/open
```

This atomically checks availability, appends `RECONCILE_SHOWN`, and returns tasks.

### Option C — GET causes the shown side effect

Simple, but violates expected HTTP semantics and may misfire because of prefetching/retries.

GPT preference: Option B. `POST /reconcile/open` represents a stateful product command and avoids pretending a side effect is a read.

## Open Reconcile

Recommended contract:

```txt
POST /api/v1/reconcile/open
```

Response:

```json
{
  "localDate": "2026-07-12",
  "timezone": "Asia/Tehran",
  "tasks": [
    {
      "id": "018f...",
      "goal": null,
      "title": "Read chapter 3",
      "plannedForDate": "2026-07-11",
      "carryCount": 1
    }
  ]
}
```

Atomic behavior:

- calculate user local date
- check unresolved eligible tasks
- check no `RECONCILE_SHOWN`, `RECONCILE_SKIPPED`, or `RECONCILE_COMPLETED` for local day
- append `RECONCILE_SHOWN`
- return the task snapshot

If unavailable:

```txt
409 RECONCILE_NOT_AVAILABLE
```

or possibly `200` with `available: false`.

GPT preference: use `200 available:false` for harmless absence and `409` only for a stale race where the client attempted to open after another request already opened it.

## Mark Reconcile started

```txt
POST /api/v1/reconcile/start
```

Response:

```txt
204 No Content
```

The endpoint appends `RECONCILE_STARTED` once for the local day.

Open question:

Is a separate start endpoint worth it, or should the first task decision imply start?

GPT preference: if `RECONCILE_STARTED` remains in the accepted event model, explicitly mark it when the user begins interacting. Inferring from the first resolution misses users who engage and then leave before deciding.

## Resolve one task

Proposed unified endpoint:

```txt
POST /api/v1/reconcile/tasks/{taskId}/resolve
```

Done request:

```json
{
  "decision": "DONE"
}
```

Carry request:

```json
{
  "decision": "CARRY",
  "plannedForDate": "2026-07-12"
}
```

Drop request:

```json
{
  "decision": "DROP"
}
```

Response:

```json
{
  "task": {
    "id": "018f...",
    "status": "ACTIVE",
    "plannedForDate": "2026-07-12",
    "carryCount": 2
  },
  "remainingCount": 2
}
```

Rules:

- task must belong to current user
- task must still be ACTIVE
- task must have been Reconcile-eligible
- CARRY requires a date that is not earlier than current local date
- domain mutation and event append are atomic
- `carryCountAtEvent` is recorded

Open question:

Should Done, Carry, and Drop use one endpoint or separate action endpoints?

Separate alternative:

```txt
POST /api/v1/tasks/{id}/complete
POST /api/v1/tasks/{id}/carry
POST /api/v1/tasks/{id}/drop
```

GPT preference: separate domain action endpoints are simpler to validate and document, while a unified Reconcile resolve endpoint keeps the frontend flow compact and can return `remainingCount`. Claude should choose based on transaction clarity, not aesthetic REST purity.

## Skip Reconcile

```txt
POST /api/v1/reconcile/skip
```

Request:

```json
{}
```

Response:

```txt
204 No Content
```

Behavior:

- append `RECONCILE_SKIPPED`
- include unresolved count in metadata
- do not modify tasks
- prohibit same-day re-trigger

Idempotency:

Repeating skip after the same-day skip should return 204 without duplicating the event.

## Complete Reconcile

```txt
POST /api/v1/reconcile/complete
```

Response:

```txt
204 No Content
```

Rules:

- append `RECONCILE_COMPLETED`
- only valid when no tasks from the opened Reconcile set remain unresolved
- repeated completion is idempotent

Open question:

Without a persistent session or snapshot ID, how does the backend know which opened task set must be resolved before completion?

Possible approaches:

### Option A — current query truth

Completion is allowed when no currently eligible unresolved tasks remain.

Risk: tasks created or backdated during the flow can change the set.

### Option B — no explicit complete endpoint

When the final eligible task is resolved, backend appends `RECONCILE_COMPLETED` automatically.

Risk: skip/open semantics and UI completion transition must be carefully coordinated.

### Option C — return an opaque reconcileAttemptId without a persistent model

Use a signed/temporary identifier or event correlation ID.

This may quietly recreate a session model through the back door.

GPT preference: Option B. Automatically append completion when resolving the final eligible task. This avoids a fragile client-issued complete command and does not require a session model.

Claude should review whether this remains correct when tasks are modified concurrently.

---

# App Open Event

The data model requires `APP_OPENED` for retention analysis.

Potential endpoint:

```txt
POST /api/v1/events/app-opened
```

Response:

```txt
204 No Content
```

Open questions:

1. Should this event be sent on every browser load, every authenticated app bootstrap, or once per app session?
2. How is duplicate noise prevented?
3. Should the frontend directly call an events endpoint?

GPT preference:

- do not expose a generic public event-ingestion endpoint
- expose a narrow product command such as `/app/open`
- record once per authenticated app session
- define app-session semantics in the validation/analytics plan

Proposed endpoint:

```txt
POST /api/v1/app/open
```

Potential response could combine bootstrap information:

```json
{
  "user": {},
  "today": {},
  "reconcileAvailable": true
}
```

However this risks becoming a large backend-for-frontend endpoint.

Claude should review whether `APP_OPENED` should be recorded as part of `GET /users/me`, a dedicated command, or another bootstrap flow.

---

# Concurrency and Idempotency

## Client-generated idempotency keys

Open question:

Do Phase 1 mutation endpoints need an `Idempotency-Key` header?

Candidates:

```txt
OTP request
create task
create goal
Reconcile decisions
onboarding complete
```

GPT preference:

- do not implement a generic idempotency-key platform for all endpoints
- make naturally repeatable commands idempotent through state checks
- use database constraints and transaction logic
- consider a client mutation ID only for actions where duplicate creation from retries is realistic

Potential field for creates:

```json
{
  "clientMutationId": "uuid"
}
```

But this adds persistence/uniqueness complexity and may be unnecessary for the closed validation.

## Optimistic locking

Open question:

Should Task include a `version` field for optimistic concurrency?

Risk without it:

- two tabs edit the same task
- stale Reconcile decision races with a normal task edit

GPT preference: JPA optimistic locking is reasonable but may be unnecessary for Phase 1. Terminal state checks and atomic update predicates may be enough:

```sql
UPDATE task
SET ...
WHERE id = :id
  AND user_id = :userId
  AND status = 'ACTIVE'
```

The affected-row count determines stale/conflict behavior.

Claude should review whether a version column earns its cost.

---

# Validation Rules

Suggested limits:

```txt
Goal title: 1–120 characters
Task title: 1–240 characters
Display name: 1–80 characters
```

These are proposed implementation constraints, not product research conclusions.

String handling:

- trim leading/trailing whitespace
- reject blank-after-trim
- preserve internal Unicode text
- do not silently truncate

Dates:

- parse strict ISO local dates
- reject invalid calendar dates
- do not accept timestamps for `plannedForDate`

UUIDs:

- malformed UUID path values return 400
- well-formed but missing/non-owned resources return 404

Open question:

Should these limits be decided now or left to the implementation plan?

GPT preference: decide them in the final API spec so frontend and backend validation agree from scaffold day one.

---

# HTTP Status Summary

Proposed conventions:

```txt
200 OK            successful read/update/action with body
201 Created       resource created
202 Accepted      OTP send accepted
204 No Content    successful command without body
400 Bad Request   malformed JSON, invalid path value, validation failure
401 Unauthorized  missing/invalid/expired/revoked session
403 Forbidden     authenticated but operation forbidden
404 Not Found     missing or non-owned resource
409 Conflict      stale state or invalid state transition
422 Unprocessable Content  optional alternative for semantic validation
429 Too Many Requests      OTP/request rate limit
500 Internal Server Error  unexpected server failure
503 Service Unavailable    temporary SMS provider failure, if exposed this way
```

Open question:

Should semantic validation use 400 or 422?

GPT preference: use 400 consistently for request validation and 409 for state conflicts. Avoid splitting validation behavior across 400/422 unless the team has a strong convention already.

---

# Proposed Endpoint Inventory

```txt
POST   /api/v1/auth/otp/request
POST   /api/v1/auth/otp/verify
POST   /api/v1/auth/logout
POST   /api/v1/auth/logout-all               optional

GET    /api/v1/users/me
PATCH  /api/v1/users/me

POST   /api/v1/onboarding/start
POST   /api/v1/onboarding/complete

POST   /api/v1/goals
GET    /api/v1/goals
GET    /api/v1/goals/{goalId}
PATCH  /api/v1/goals/{goalId}
DELETE /api/v1/goals/{goalId}                probably excluded

POST   /api/v1/tasks
GET    /api/v1/tasks
GET    /api/v1/tasks/{taskId}
PATCH  /api/v1/tasks/{taskId}
POST   /api/v1/tasks/{taskId}/complete

GET    /api/v1/today

POST   /api/v1/reconcile/open
POST   /api/v1/reconcile/start
POST   /api/v1/reconcile/tasks/{taskId}/resolve
POST   /api/v1/reconcile/skip
POST   /api/v1/reconcile/complete            possibly unnecessary

POST   /api/v1/app/open                      under review
```

This is a proposal, not the final API list.

---

# Deliberately Excluded Endpoints

```txt
/admin
/analytics
/ai
/chat
/recommendations
/routines
/habits
/calendar
/time-blocks
/deadlines
/notifications
/teams
/workspaces
/tags
/priorities
/oauth
/password
/auth/refresh
```

No generic client event ingestion endpoint is proposed.

---

# Questions for Claude

Please review this as a Phase 1 implementation contract, not as a generic REST style exercise.

1. Which proposed endpoints are unnecessary or missing for the accepted Day-0, Today, Task, Goal, Auth, and Reconcile flows?
2. Should `/auth/me` exist, or should `/users/me` be the only current-user endpoint?
3. Should OTP request return 202 with `resendAvailableAt`, or 204?
4. Should OTP verify return `isNewUser`, or should onboarding state alone drive routing?
5. Should task responses embed a minimal Goal summary or only `goalId`?
6. Does Phase 1 need pagination for goals or tasks?
7. Should generic task PATCH own planning/unscheduling events, or should these be dedicated action endpoints?
8. Should terminal actions be idempotent?
9. How should Drop + Undo be reconciled with the accepted no-reopen data-model rule?
10. Should Reconcile opening be a POST command rather than a GET with side effects?
11. Is a separate `RECONCILE_STARTED` endpoint justified?
12. Should Reconcile use one unified resolve endpoint or separate Done/Carry/Drop endpoints?
13. Without a persistent ReconcileSession, should completion be automatic after the final resolution rather than client-triggered?
14. How should the opened Reconcile task set behave if tasks change concurrently?
15. Where should `APP_OPENED` be recorded without exposing a generic event API or creating duplicate noise?
16. Are idempotency keys or optimistic locking necessary for this Phase 1 threat/concurrency model?
17. Are the proposed Problem Details fields and error codes sufficient and not excessive?
18. Should semantic validation use 400 consistently, or 422?
19. Which validation limits should be fixed in the API spec?
20. What is the smallest revision required before this becomes `04-Specs/api-contracts-phase-1.md`?

## Requested Review Style

Please be critical and concrete:

- identify API complexity that does not support Phase 1 validation
- identify frontend/backend ambiguity
- identify race conditions and retry problems
- distinguish product commands from generic CRUD
- preserve accepted domain and UX decisions
- propose exact endpoint/body/status replacements where you disagree

---

# Expected Outcome

After Claude review and Mahdi approval, formalize:

```txt
04-Specs/api-contracts-phase-1.md
```

The final spec should own:

```txt
endpoint inventory
request and response DTOs
status codes
problem-details contract
stable error codes
validation limits
resource ownership rules
idempotency behavior
Reconcile command boundaries
Drop + Undo behavior
APP_OPENED boundary
```

The final API spec must not reopen the product scope, data model, authentication method, or implementation stack.