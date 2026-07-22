# AI-Native MVP Implementation Stack

## Status

`AMENDED — PROVISIONAL UNTIL M1 SLICE LOCK`

## Retained baseline

### Frontend

- React + strict TypeScript + Vite
- versioned REST client and explicit state machines
- accessible form, review, confirmation, failure, cancellation, stale/conflict, and degraded states
- unit/component/contract/E2E testing

### Backend

- Java + Spring Boot
- package-by-feature domain/application boundaries
- PostgreSQL + versioned migrations
- explicit DTO/schema validation
- RFC 9457 Problem Details and trace identity
- transaction, concurrency, idempotency, outbox, and observability support
- integration tests with real PostgreSQL behavior

### Infrastructure

- Dockerized services
- Nginx retained as the edge/deployment baseline
- health/ready checks, external monitoring, structured logs, traces, alerts, backups, rollback, and incident procedures

## AI runtime requirements

- separate Planning and Reconcile ports;
- bounded/versioned context builders and Context Scope Manifest;
- no model tools;
- pinned provider/model/prompt/schema/policy artifacts;
- strict structured-output gates;
- bounded retry, timeout, rate, circuit, token, and spend controls;
- cancellation, late-result discard, kill switches, and deterministic mock provider;
- restricted safety/privacy observability.

## Deliberately excluded

- PWA/offline-first mutation sync;
- model tool/function calling;
- automatic provider fallback during pilot;
- refresh-token/per-device sessions;
- calendar, exact time blocks, and broad integrations.

## Values required before scaffold lock

Exact supported versions, build manifests, migration numbering, CI images, dependency-security policy, owners, and environment configuration must be selected and recorded before M1 is `SLICE_LOCKED`. Provider and operational values follow their later milestone gates in [[02-Decisions/repository-artifact-migration-ledger]].

Authority: `LEG-02`, `LEG-07`, Discussions 019A–020C, and provisional sequencing in Discussion 022.
