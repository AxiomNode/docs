# Contract Governance

Last updated: 2026-05-03.

## Purpose

This capability keeps request, response, and event semantics aligned across repositories through canonical schema sources and shared generated integration artifacts.

## Functional objective

- maintain one source of truth for HTTP and payload contracts
- reduce integration drift between services, BFFs, and shared clients
- provide repeatable SDK generation and validation across the platform

## Ownership and boundaries

### Main owners

- `contracts-and-schemas`
- `shared-sdk-client`

### Explicit boundaries

- owns OpenAPI and JSON Schema sources in `contracts-and-schemas`
- owns shared SDK artifacts and reusable route/schema helpers in `shared-sdk-client`
- delegates runtime behavior and rollout timing to consuming repositories
- does not itself own service deployment semantics or runtime persistence

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| Source artifact | versioned OpenAPI specs under `contracts-and-schemas/schemas/openapi/` | HTTP contract source of truth |
| Source artifact | JSON Schemas under `contracts-and-schemas/schemas/json/` | payload and query semantics |
| Generated artifact | shared SDK modules under `shared-sdk-client/typescript/src/` | consumer integration surface |

### Inputs

- API design changes and schema updates
- examples and validation fixtures in the contracts repo
- consumer feedback and breakage signals from runtime repositories

### Outputs

- canonical specs and schemas
- generated SDK exports such as `AiEngineClient`, `createGameRoutes()`, and reusable validators
- CI and compile feedback from consumers

### Contract sources

- OpenAPI specs such as `internal-ai-engine.v1.yaml`, `internal-microservice-quizz.v1.yaml`, `internal-microservice-wordpass.v1.yaml`, and `internal-microservice-users.v1.yaml`
- JSON Schemas such as `game-generate.request.v1.json`, `random-game.query.v1.json`, and `leaderboard.query.v1.json`
- `shared-sdk-client/typescript/src/ai-engine-client.ts` and `game-routes.ts`

## Main functional flow

1. A contract change lands in `contracts-and-schemas`.
2. Schema validation confirms the source artifacts remain internally consistent.
3. `shared-sdk-client` is regenerated or manually updated to reflect the new contract shape.
4. Runtime repositories compile and test against the updated shared artifact set.

## Persistence and runtime state

### Owned persistence

- versioned schema files in the contracts repo
- generated SDK outputs and reusable source modules in `shared-sdk-client`

### Runtime state

- no direct runtime state ownership, but strong influence on cross-repo compatibility and rollout safety

### State risks

- source contract and generated SDK can drift if regeneration is skipped
- consumers may lag behind the latest compatible artifact set
- runtime repositories can keep local validators that slowly diverge from the central contract intent

## Observability evidence

### Health signals

- schema validation and generation checks succeeding

### Operational signals

- shared SDK updates
- compile/test feedback in consuming repos
- service route or payload regressions after contract changes

### Verification evidence

- schema validation in CI
- consumer integration checks in quiz, word-pass, gateway, and BFF repos
- cross-repo build success after contract changes

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Contract drift | schema updated without SDK or consumer update | compile/runtime mismatch or silent behavior skew | CI failures, integration errors | regenerate SDK and align consumers |
| Incompatible active change | breaking contract in a live path | request or response failures in consumers | consumer regression and smoke tests | version route/schema and roll forward safely |
| Shared-helper drift | helper code evolves without source schema change | route behavior differs from documented contract | consumer tests and review findings | reconcile helpers with canonical contract source |

## Security and access model

- contract source repos should be tightly write-controlled because they affect many downstream systems
- generated artifacts must not embed secrets or environment-specific configuration
- schema evolution should preserve validation boundaries that protect runtime services from malformed input

## Deployment and rollback implications

### Deployment impact

- contract work often requires coordinated updates across source repo, SDK repo, and consuming services
- rollout ordering matters when consumers expect regenerated helpers or schemas

### Rollback impact

- rollback must respect schema/version compatibility, not just package publication state
- reverting only one side of the source/SDK pair can leave consumers in a broken state

## Related capabilities

- [AI content generation](../ai/ai-content-generation.md)
- [Public edge routing](../edge/public-edge-routing.md)
- [Quiz inventory and generation](../domain/quiz-inventory-and-generation.md)
- [Player identity and profile](../player/player-identity-and-profile.md)

## Open questions

- which contracts are still implicitly owned in service repos instead of centrally versioned
- whether every consuming repo updates shared artifacts with enough discipline after contract changes

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
- [Repository standards adoption guide](../../governance/repository-standards-adoption.md)
