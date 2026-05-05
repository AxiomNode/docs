# Mobile Gameplay Orchestration

Last updated: 2026-05-03.

## Purpose

This capability turns quiz and word-pass inventory plus player state into a mobile-facing gameplay flow with stable request shapes and controlled synchronization back to the platform.

## Functional objective

- expose mobile-friendly quiz and word-pass routes through one BFF
- shield the client from downstream service topology and schema complexity
- coordinate random retrieval, generation, profile sync, and event upload for gameplay sessions

## Ownership and boundaries

### Main owners

- `bff-mobile`
- `mobile-app`

### Downstream dependencies

- `microservice-quizz`
- `microservice-wordpass`
- `microservice-users`

### Explicit boundaries

- owns mobile route composition in `bff-mobile/src/app/routes/mobile.ts`
- owns player-facing payload shaping and request validation at the mobile BFF layer
- delegates content persistence and generation semantics to the domain services and player identity semantics to `microservice-users`
- does not expose the domain services directly to the mobile client

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | `GET /v1/mobile/games/quiz/random` | random quiz retrieval through `bff-mobile/src/app/routes/mobile.ts` |
| HTTP route | `GET /v1/mobile/games/wordpass/random` | random word-pass retrieval |
| HTTP route | `POST /v1/mobile/games/quiz/generate` | quiz generation request orchestration |
| HTTP route | `POST /v1/mobile/games/wordpass/generate` | word-pass generation request orchestration |
| HTTP route | mobile profile and sync routes | player session and event synchronization |

### Inputs

- player auth/session context forwarded from the mobile client
- category, language, and count filters for random inventory routes
- gameplay generation payloads validated with shared schemas such as `BaseGenerateSchema`
- event-batch payloads and profile updates synchronized through the player service

### Outputs

- mobile-ready game sessions and random inventory payloads
- player sync acknowledgements and profile snapshots
- downstream calls to quiz, word-pass, and users microservices

### Contract sources

- `bff-mobile/src/app/routes/mobile.ts`
- shared SDK route schemas such as `RandomGameQuerySchema`, `BaseGenerateSchema`, `GameEventsSyncSchema`, and `PlayerProfileUpdateSchema`
- integration coverage in `bff-mobile/src/tests/mobile.test.ts`

## Main functional flow

1. The mobile client sends a gameplay request through `api-gateway` to `bff-mobile`.
2. `bff-mobile` validates the route payload and picks the correct downstream service based on game type.
3. Quiz or word-pass content is retrieved from random inventory or requested from generation flows in the domain services.
4. Player profile or event synchronization is forwarded to the users service.
5. `bff-mobile` returns a mobile-shaped response while the client maintains local gameplay continuity.

## Persistence and runtime state

### Owned persistence

- mobile local session and offline continuity state in the client

### Runtime state

- BFF request validation and downstream orchestration state
- player connectivity state handled by the client and mobile sync paths
- no ownership of quiz or word-pass inventory persistence

### State risks

- offline and online state diverging during delayed event synchronization
- client-visible payload drift when BFF mappings fall behind domain contracts
- partial sync creating ghost or repeated gameplay events

## Observability evidence

### Health signals

- `bff-mobile` health plus gateway mobile route health

### Operational signals

- mobile BFF route failures and downstream error responses
- sync payload results and profile update acknowledgements
- gateway traces through `/v1/mobile/*`

### Verification evidence

- `bff-mobile/src/tests/mobile.test.ts`
- `bff-mobile/src/tests/health.test.ts`
- platform workflow validations referenced in `docs/guides/validation/primary-workflows-validation.md`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Invalid mobile payload shape | BFF mapping drift or contract change downstream | app cannot render or play returned game | mobile integration tests, client errors | fix the BFF response shape or align shared schemas |
| Offline sync divergence | delayed uploads or duplicate event replay | wrong player stats or inconsistent progress | sync responses, profile comparison | harden idempotency and reconcile uploaded events |
| Partial downstream outage | one game service unavailable while others are healthy | only some game modes fail from mobile | BFF route logs and panelized diagnostics | degrade gracefully per route and restore the affected service |

## Security and access model

- mobile clients do not talk directly to the domain services or ai-engine
- player identity and profile semantics are delegated to protected backend services
- local device state is not a trusted source for server-side profile and leaderboard truth

## Deployment and rollback implications

### Deployment impact

- mobile behavior depends on compatibility across `mobile-app`, `api-gateway`, and `bff-mobile`
- contract changes can require coordinated BFF and client updates

### Rollback impact

- server rollback can break a newer mobile client if payload shape changes were not versioned or shielded by the BFF
- client rollback does not restore already diverged server-side sync state

## Related capabilities

- [Public edge routing](../edge/public-edge-routing.md)
- [Player identity and profile](./player-identity-and-profile.md)
- [Quiz inventory and generation](../domain/quiz-inventory-and-generation.md)
- [Word-pass inventory and generation](../domain/wordpass-inventory-and-generation.md)

## Open questions

- how much offline conflict-handling is formalized in platform docs versus only implemented in the app and BFF
- whether every critical mobile gameplay path has an end-to-end smoke check across gateway, BFF, and domain services

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Mobile app development (Firebase)](../../experience/mobile-app-development-firebase.md)
- [Primary validation workflows](../../validation/primary-workflows-validation.md)
