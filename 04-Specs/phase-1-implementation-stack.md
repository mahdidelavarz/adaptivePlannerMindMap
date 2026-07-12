# Phase 1 Implementation Stack

## Status

Accepted implementation baseline for Phase 1.

## Purpose

This file turns [[02-Decisions/ADR-002-phase-1-technical-foundation]] into an implementation-ready stack.

It defines the frameworks, libraries, packages, build tools, testing tools, and deployment baseline the team should use when scaffolding the frontend and backend.

Version policy:

- pin exact versions in `package.json`, `package-lock.json`, `pom.xml`, and Docker image tags when scaffolding
- stay inside the major versions listed here during Phase 1
- do not upgrade major versions during the 14-day validation unless a security or blocking compatibility issue requires it

---

# 1. Repository Shape

```txt
adaptive-planner/
  frontend/
  backend/
  infra/
    nginx/
    docker/
  docs/
```

Use one Git repository with two independent build systems:

```txt
frontend: npm
backend: Maven Wrapper
```

Phase 1 does not require Nx, Turborepo, or another monorepo framework.

---

# 2. Frontend Stack

## Runtime and build

```txt
Node.js: current LTS supported by Vite 8
Package manager: npm
React: 19.2.x
TypeScript: 5.x with strict mode
Vite: 8.x
```

Required packages:

```txt
react
react-dom
typescript
vite
@vitejs/plugin-react
```

TypeScript rules:

```txt
strict: true
noUncheckedIndexedAccess: true
exactOptionalPropertyTypes: true
noImplicitOverride: true
```

Do not use Next.js in Phase 1.

## Routing

```txt
React Router: 8.x
Mode: client-side browser router
```

Package:

```txt
react-router
```

Use `createBrowserRouter` with route-level lazy loading.

Minimum route groups:

```txt
/auth
/onboarding
/today
/tasks
/goals
/reconcile
/settings
```

## Server state

```txt
TanStack Query: 5.x
```

Packages:

```txt
@tanstack/react-query
@tanstack/react-query-devtools
```

Use TanStack Query for API reads, mutations, cache invalidation, loading/error state, and safe read retries.

Do not duplicate API response state inside Zustand.

## Client-only state

```txt
Zustand: 5.x
```

Package:

```txt
zustand
```

Use Zustand only for:

- temporary auth UI state
- onboarding draft state before submission
- local filters
- non-server UI preferences

Do not use Redux in Phase 1.

## HTTP client

```txt
Axios: 1.x
```

Package:

```txt
axios
```

Create one configured API client with:

- `/api/v1` base path
- `withCredentials: true` when cookie-based JWT transport is used
- normalized API error mapping
- request timeout
- refresh handling only if refresh tokens are implemented

Do not call Axios directly from components. API calls belong in feature API modules.

## Forms and validation

```txt
React Hook Form: 7.x
Zod: 4.x
```

Packages:

```txt
react-hook-form
zod
@hookform/resolvers
```

Use React Hook Form for form state and Zod for frontend input validation.

Frontend validation improves UX but never replaces backend validation.

## Styling and design system

```txt
Tailwind CSS: 4.x
Radix UI primitives
Class Variance Authority
```

Packages:

```txt
tailwindcss
@tailwindcss/vite
@radix-ui/react-dialog
@radix-ui/react-dropdown-menu
@radix-ui/react-popover
@radix-ui/react-toast
@radix-ui/react-tooltip
@radix-ui/react-tabs
@radix-ui/react-checkbox
@radix-ui/react-radio-group
@radix-ui/react-switch
@radix-ui/react-slot
class-variance-authority
clsx
tailwind-merge
```

Install only the Radix primitives actually used by the UI.

Component conventions:

- primitives under `src/components/ui`
- feature components under `src/features/<feature>`
- no MUI or Ant Design in Phase 1

## Icons

```txt
@iconify/react
```

Use a limited icon collection consistently.

## Dates and timezone

Packages:

```txt
date-fns
date-fns-tz
```

Rules:

- backend timestamps are UTC ISO-8601 instants
- `plannedForDate` is `YYYY-MM-DD`
- default Phase 1 timezone is `Asia/Tehran`
- frontend sends the detected IANA timezone during profile creation
- never hardcode `UTC+03:30`
- Reconcile uses the user's local date

## PWA

```txt
vite-plugin-pwa
```

Phase 1 responsibilities:

- web manifest
- installable shell
- icons
- basic static-asset caching
- update notification strategy

Excluded:

- offline task mutation
- background sync
- offline-first storage
- conflict resolution
- local event queues

## Toasts

```txt
sonner
```

Use it for concise feedback such as:

```txt
Dropped. Undo
Saved
Could not connect
```

Do not use toasts instead of visible form errors.

## Frontend testing

```txt
Vitest: 4.x
React Testing Library
MSW
Playwright
```

Packages:

```txt
vitest
@vitest/coverage-v8
jsdom
@testing-library/react
@testing-library/jest-dom
@testing-library/user-event
msw
@playwright/test
```

Mandatory Playwright flows:

```txt
phone OTP login with mocked SMS provider
new-user Day-0 onboarding
empty Today path
create standalone task
create goal-linked task
Reconcile Done
Reconcile Carry
Reconcile Drop + Undo
Not now / skipped reconcile
no same-day Reconcile re-trigger
next-local-day Reconcile re-trigger
```

## Frontend quality tooling

```txt
eslint
@eslint/js
typescript-eslint
eslint-plugin-react-hooks
eslint-plugin-react-refresh
prettier
prettier-plugin-tailwindcss
```

Rules:

- ESLint errors fail CI
- TypeScript errors fail CI
- formatting is automated
- no `any` without a localized justification

---

# 3. Backend Stack

## Runtime and build

```txt
Java: 21 LTS
Spring Boot: 4.1.x
Build tool: Maven Wrapper
Packaging: executable JAR
Web model: Spring MVC / servlet stack
```

Required base dependencies:

```txt
spring-boot-starter-webmvc
spring-boot-starter-validation
spring-boot-starter-data-jpa
spring-boot-starter-security
spring-boot-starter-oauth2-resource-server
spring-boot-starter-actuator
```

`spring-boot-starter-oauth2-resource-server` remains only for Spring Security JWT encoder/decoder support for application-issued JWTs.

Do not add `spring-boot-starter-oauth2-client`; Google OAuth is not part of Phase 1.

Do not use WebFlux.

## Persistence

```txt
PostgreSQL: 17
Hibernate/JPA: managed by Spring Boot
Database migrations: Flyway
```

Dependencies:

```txt
org.postgresql:postgresql
org.flywaydb:flyway-core
org.flywaydb:flyway-database-postgresql
```

Rules:

- Flyway owns schema changes
- Hibernate schema auto-generation is disabled outside tests
- `ddl-auto=validate` in production
- deployed migrations are immutable
- database constraints are mandatory

## DTO mapping

Use Java records for immutable request/response DTOs where appropriate.

Use explicit manual mapping functions in Phase 1.

Do not add MapStruct initially. Add it later only if mapping volume becomes repetitive enough to justify another annotation processor.

Never expose JPA entities directly from controllers.

## Authentication and security

Official Phase 1 auth:

```txt
phone-number OTP
JWT-based application sessions
password recovery/reset through verified phone OTP
```

There is no Google OAuth, email login, or password login in Phase 1.

Dependencies:

```txt
spring-boot-starter-security
spring-boot-starter-oauth2-resource-server
```

Use Spring Security's Nimbus-based JOSE/JWT support for application-issued tokens.

Recommended token transport:

```txt
HttpOnly
Secure
SameSite cookie
```

The auth implementation must explicitly define before coding:

```txt
frontend origin
API origin
Nginx /api proxy path
CORS allowed origins
CORS credentials policy
cookie Domain
cookie Path
cookie SameSite value
cookie Secure behavior in local and production
access-token cookie name and lifetime
refresh-token cookie name, lifetime, rotation, and revocation if refresh tokens are used
CSRF strategy for cookie-authenticated state-changing requests
```

Auth package responsibilities:

- phone normalization
- OTP request
- OTP verification
- user creation/authentication
- JWT issuance
- refresh/revocation if implemented
- password recovery via verified phone OTP

Items Mahdi will define before auth implementation:

```txt
OTP expiration
resend cooldown
maximum verification attempts
request rate limit
single-use OTP invalidation
```

These remain reminders only; exact values are not defined here.

## OTP provider adapter

Application-owned interface:

```java
public interface OtpDeliveryGateway {
    void send(String normalizedPhoneNumber, String code);
}
```

Provider-specific code stays behind an adapter.

Use Spring `RestClient` for provider HTTP calls unless the provider SDK is clearly simpler and maintainable.

Do not call the SMS provider directly from controllers or domain services.

## JWT claims

Required claims:

```txt
sub: internal user id
iss: application issuer
iat
exp
jti
roles or authorities when needed
```

Do not put private profile data into JWT claims.

## Domain organization

Use package-by-feature:

```txt
com.adaptiveplanner
  auth
  user
  goal
  task
  reconcile
  event
  common
  config
```

Each feature may contain:

```txt
api
application
domain
infrastructure
```

Keep dependency direction clear without building a ceremonial clean-architecture maze.

## Transactions and events

Use Spring `@Transactional` on application-service commands.

State mutation and behavioral-event append must occur in the same PostgreSQL transaction.

Examples:

```txt
complete task + task_completed
carry task + task_carried
drop task + task_dropped
skip reconcile + reconcile_skipped
```

Detailed event fields belong to [[04-Specs/data-model-phase-1]].

## Validation and errors

Use:

```txt
Jakarta Bean Validation
Problem Details for HTTP APIs
@RestControllerAdvice
```

Do not leak stack traces or database errors.

## API documentation

```txt
org.springdoc:springdoc-openapi-starter-webmvc-ui
```

Expose Swagger UI only in development/test or protect it in production.

## Logging and health

Use Spring Boot defaults:

```txt
SLF4J + Logback
Spring Boot Actuator
```

Minimum endpoints:

```txt
/actuator/health
/actuator/info
```

Log:

- request correlation/trace ID
- authentication failures without secrets or OTP values
- SMS-provider failures
- unexpected Reconcile transaction failures

Never log:

- OTP values
- JWT values
- refresh tokens

## Backend testing

Dependencies:

```txt
spring-boot-starter-test
spring-security-test
spring-boot-testcontainers
org.testcontainers:junit-jupiter
org.testcontainers:postgresql
org.wiremock:wiremock
```

Mandatory integration tests:

```txt
task update rolls back when event insert fails
event insert rolls back when task update fails
Reconcile selects only unresolved tasks before the user's local date
Asia/Tehran day-boundary behavior
normalized phone uniqueness
OTP request and verification flow with mocked provider
OTP single-use behavior after Mahdi defines the rule
cookie-authenticated API access
CORS credentials behavior through the configured API boundary
```

---

# 4. Database Baseline

```txt
PostgreSQL 17
UTF-8
UTC timestamps
IANA user timezone
ISO local DATE for plannedForDate
```

Core extension:

```txt
pgcrypto
```

Use one UUID-generation strategy consistently.

Initial indexes should cover:

```txt
users(normalized_phone) unique
tasks(user_id, status, planned_for_date)
tasks(goal_id)
events(user_id, occurred_at)
events(user_id, type, occurred_at)
events(entity_type, entity_id, occurred_at)
```

No `user_identities` table is needed in Phase 1. The normalized phone number is the single authentication identity anchor.

Exact schema belongs to [[04-Specs/data-model-phase-1]].

---

# 5. Infrastructure and Deployment

## Containers

Use Docker Compose for development and VPS deployment.

Services:

```txt
frontend
backend
postgres
nginx
```

Use multi-stage Dockerfiles.

## Nginx

Responsibilities:

- TLS termination
- static frontend serving
- `/api` proxy to Spring Boot
- cookie-preserving reverse proxy behavior
- security headers
- compression
- SPA fallback

Prefer a same-site deployment shape:

```txt
https://app-domain/
https://app-domain/api/
```

This reduces CORS and cookie complexity compared with separate frontend and API domains.

## Configuration

Spring profiles:

```txt
local
test
production
```

Required secret categories:

```txt
JWT signing material
OTP/SMS provider credentials
database credentials
```

Never commit secrets.

## CI

Frontend:

```txt
npm ci
npm run lint
npm run typecheck
npm run test
npm run build
```

Backend:

```txt
./mvnw verify
```

Integration tests use PostgreSQL through Testcontainers.

---

# 6. Deliberately Excluded

Do not add without a concrete Phase 1 requirement:

```txt
Google OAuth
spring-boot-starter-oauth2-client
MapStruct
user_identities polymorphic auth table
Redux
Next.js
GraphQL
WebSockets
Kafka/RabbitMQ
Redis
Elasticsearch
Kubernetes
Temporal
offline sync libraries
AI SDKs
vector databases
microservice libraries
```

A package is not progress merely because npm or Maven successfully downloaded it.

---

# 7. Scaffold Checklist

## Frontend

- [ ] Create Vite React TypeScript project
- [ ] Enable strict TypeScript rules
- [ ] Configure React Router
- [ ] Configure TanStack Query
- [ ] Configure Zustand conventions
- [ ] Configure Axios with cookie credentials
- [ ] Configure React Hook Form + Zod
- [ ] Configure Tailwind CSS 4
- [ ] Add only required Radix primitives
- [ ] Configure Iconify
- [ ] Configure timezone/date utilities
- [ ] Configure PWA plugin
- [ ] Configure Vitest, Testing Library, MSW, and Playwright
- [ ] Configure ESLint and Prettier

## Backend

- [ ] Create Java 21 / Spring Boot 4.1 Maven project
- [ ] Add Web MVC, Validation, JPA, Security, Resource Server, and Actuator
- [ ] Configure PostgreSQL and Flyway
- [ ] Configure package-by-feature structure
- [ ] Configure manual DTO mappers
- [ ] Configure global Problem Details errors
- [ ] Configure OpenAPI
- [ ] Configure OTP provider port/adapter
- [ ] Configure JWT encoder/decoder
- [ ] Configure cookie, CORS, CSRF, and Nginx auth boundary
- [ ] Configure `Asia/Tehran` default timezone behavior
- [ ] Configure Testcontainers and integration tests
- [ ] Add atomic mutation/event transaction tests

## Infrastructure

- [ ] Add Dockerfiles
- [ ] Add Docker Compose
- [ ] Add Nginx config
- [ ] Define same-site frontend/API routing
- [ ] Define environment templates without secrets
- [ ] Add GitHub Actions checks
- [ ] Pin container image versions

---

# 8. Next Step

Use this stack while creating:

```txt
04-Specs/data-model-phase-1.md
```

The Data Model spec should define entities, constraints, events, indexes, and transaction behavior without reopening framework or package choices.