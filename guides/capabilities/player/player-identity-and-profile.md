# Player Identity And Profile

Last updated: 2026-05-03.

## Purpose

This capability associates gameplay with an authenticated player identity and exposes profile, session, event, and leaderboard semantics to the mobile and operator channels.

## Functional objective

- validate player identity and session information
- persist profile and gameplay-event state in the users service
- expose leaderboard and profile reads to BFF consumers

## Ownership and boundaries

### Main owners

- `microservice-users`

### Supporting owners

- `bff-mobile`
- `bff-backoffice`

### Downstream dependencies

- PostgreSQL via Prisma
- Firebase token validation and environment auth policy

### Explicit boundaries

- owns player profile, event persistence, session sync, and leaderboard aggregation in `microservice-users/src/app/routes/users.ts` and `userService.ts`
- owns Firebase-backed identity validation in `firebaseAuthService.ts`
- delegates public/mobile ingress to BFFs and edge auth forwarding to upstream layers
- does not own gameplay generation, quiz inventory, or mobile UI state

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | profile read/update routes in `microservice-users/src/app/routes/users.ts` | player-facing identity surface |
| HTTP route | session sync routes | BFF-coordinated auth/session persistence |
| HTTP route | leaderboard routes | aggregated player ranking reads |
| HTTP route | gameplay event ingestion routes | event batches update stats and history |

### Inputs

- Firebase or forwarded auth tokens
- profile mutation payloads
- gameplay event batches including game type, category, language, outcome, score, and duration

### Outputs

- player profile snapshots
- session-sync results
- leaderboard payloads and gameplay aggregates
- persisted side effects in PostgreSQL-backed tables

### Contract sources

- `microservice-users/src/app/routes/users.ts`
- `microservice-users/src/app/services/userService.ts`
- `microservice-users/src/app/services/firebaseAuthService.ts`
- tests in `microservice-users/src/tests/userService.test.ts` and `users.integration.test.ts`

## Main functional flow

1. A BFF forwards an authenticated player request or gameplay event payload to `microservice-users`.
2. The service validates auth context using Firebase-oriented logic or environment-specific fallback policy.
3. `UserService` stores or updates profile, session, and event state through Prisma.
4. Aggregated profile and leaderboard reads are exposed back to the calling BFF.

## Persistence and runtime state

### Owned persistence

- users, profiles, gameplay events, and leaderboard aggregates in PostgreSQL

### Runtime state

- auth strictness varies by environment and can influence acceptance behavior
- aggregation results depend on current event history and deduplication quality

### State risks

- duplicate gameplay events inflating score or leaderboard state
- auth-mode drift between local/dev and stricter environments
- partial writes or aggregation bugs producing inconsistent profile snapshots

## Observability evidence

### Health signals

- users service health and readiness

### Operational signals

- auth/session logs
- profile read/update traces
- leaderboard and gameplay-event counters

### Verification evidence

- `microservice-users/src/tests/userService.test.ts`
- `microservice-users/src/tests/users.integration.test.ts`
- `microservice-users/src/tests/firebaseAuthService.test.ts`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Invalid auth | bad token, missing token, or environment misconfiguration | player profile flows fail | auth logs, 401/403 responses | fix token flow or environment auth config |
| Stats inconsistency | duplicate events or aggregation defects | wrong profile, history, or leaderboard output | profile diff, leaderboard inspection | deduplicate and reprocess affected events |
| Session drift | BFF and users service hold different session assumptions | client sees stale identity or sync state | session sync checks | resync session and normalize environment policy |

## Security and access model

- player identity relies on Firebase-backed validation or tightly scoped fallback behavior
- gameplay-event payloads can contain sensitive behavioral data and should stay on protected internal surfaces
- leaderboard routes must avoid leaking unrelated user profile details

## Deployment and rollback implications

### Deployment impact

- schema and aggregation changes must preserve compatibility for BFF consumers
- auth-policy changes can alter acceptance behavior without changing route names

### Rollback impact

- application rollback does not undo already persisted events
- fixing a bad rollout may require data cleanup in addition to binary rollback

## Related capabilities

- [Mobile gameplay orchestration](./mobile-gameplay-orchestration.md)
- [Public edge routing](../edge/public-edge-routing.md)
- [Contract governance](../platform/contract-governance.md)

## Open questions

- whether duplicate-event defenses are enforced uniformly across all event-ingestion paths
- whether environment-specific auth fallback rules are documented with enough precision for operational use

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Primary validation workflows](../../validation/primary-workflows-validation.md)
- [Service communication model](../../communication/service-communication.md)
