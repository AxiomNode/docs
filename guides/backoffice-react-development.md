# Backoffice Development Guide (React)

## Goal

Define a practical blueprint to build and evolve the AxiomNode backoffice web application for operations, monitoring, and controlled data management.

## Functional scope

The backoffice covers three capability groups:

1. Administration: users, catalogs, and generated content operations.
2. Observability: service health and runtime metrics.
3. Manual remediation: controlled corrective actions when automation fails.

## Recommended architecture

Main path:

1. React backoffice -> `api-gateway`
2. `api-gateway` -> `bff-backoffice`
3. `bff-backoffice` -> domain services

Current key routes:

- `GET /v1/backoffice/users/leaderboard`
- `GET /v1/backoffice/monitor/stats`

## Design principles

1. Task-oriented UI (tables, filters, contextual actions, auditability).
2. No direct frontend calls to domain microservices.
3. Typed contracts through shared SDK utilities.
4. Feature flags for incremental rollout.

## Recommended frontend stack

- React 19+
- Vite
- Strict TypeScript
- React Router
- TanStack Query
- React Hook Form + Zod
- Component library with shared design tokens

## Suggested initial structure

```text
src/
  app/
  modules/
  shared/
  pages/
  styles/
```

## Security baseline

- Operator auth through Firebase (or equivalent) with role claims.
- Role-based authorization for privileged actions.
- Auditable manual changes (actor, action, timestamp, target, delta).
- No hardcoded secrets in frontend code.

## Local environment flow

1. Start required domain services.
2. Inject dev secrets from `secrets`.
3. Run edge integration stack from `platform-infra`.
4. Start frontend against local `api-gateway`.

## Testing baseline

- Unit tests for core UI logic.
- Integration tests for data-heavy pages.
- E2E tests for critical operational flows.
- Contract validation against real edge responses in CI.