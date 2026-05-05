# Quiz Inventory And Generation

Last updated: 2026-05-03.

## Purpose

This capability owns quiz generation orchestration, duplicate control, persistence, and random retrieval for mobile and operator consumers.

## Functional objective

- generate valid quiz models through ai-engine
- persist reusable quiz inventory and history
- serve random or filtered quiz content through domain routes
- support operator inspection, maintenance, and batch generation workflows

## Ownership and boundaries

### Main owners

- `microservice-quizz`

### Downstream dependencies

- PostgreSQL via Prisma
- `ai-engine-api`

### Explicit boundaries

- owns quiz route registration in `microservice-quizz/src/app/routes/games.ts`
- owns orchestration, duplicate detection, storage, and batch-generation logic in `generationService.ts`
- consumes ai-engine through the shared `AiEngineClient`
- does not own mobile/backoffice presentation concerns or model execution internals

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | quiz generation routes in `games.ts` | AI-assisted quiz creation |
| HTTP route | `/games/models/random` | random inventory retrieval |
| HTTP route | history, grouping, and maintenance routes | operator-facing inventory inspection |
| Service method | `GenerationService.generateAndStoreWithResult()` | core orchestration and dedup path |

### Inputs

- generation requests with category, difficulty, and count context
- inventory filters for random/history/grouped reads
- batch-generation parameters and maintenance operations
- ai-engine catalog and generation responses

### Outputs

- validated quiz payloads persisted into `gameGeneration`
- random inventory responses
- grouped summaries, history pages, and process snapshots
- ingest acknowledgements when RAG content is synchronized upstream

### Contract sources

- `microservice-quizz/src/app/routes/games.ts`
- `microservice-quizz/src/app/services/generationService.ts`
- `shared-sdk-client/typescript/src/ai-engine-client.ts`
- shared canonicalization helpers such as `canonicalizeQuizModel()`

## Main functional flow

1. A BFF or operator route calls the quiz service generation or inventory endpoint.
2. `GenerationService` resolves the requested category, item count, and difficulty.
3. The service calls ai-engine through `AiEngineClient.generate()` and records outbound request metrics through the observer path.
4. The returned payload is canonicalized, hashed into a uniqueness key, and checked against persisted content.
5. New valid models are stored in Prisma; duplicates are rejected without new rows.
6. Random and history routes serve persisted inventory back to upstream consumers.

## Persistence and runtime state

### Owned persistence

- quiz inventory and history rows in the Prisma-managed `gameGeneration` table

### Runtime state

- cached category catalog source with `local-fallback` or `ai-engine`
- AI auth circuit state with failure streak, cooldown, and opened-until timestamp
- in-memory process snapshots for background and blocking batch-generation runs
- grouped-summary cache used to avoid recomputing category totals on every request

### State risks

- stored legacy records can become invalid when validation logic evolves
- auth-circuit state can block generation after repeated upstream auth failures
- stale category cache can hide upstream recovery or degradation temporarily

## Observability evidence

### Health signals

- quiz service health and readiness

### Operational signals

- generation observer callbacks and outbound ai-engine request metrics
- grouped inventory summaries and history inspection routes
- process snapshots from batch-generation execution

### Verification evidence

- `microservice-quizz/src/tests/generationService.test.ts`
- `microservice-quizz/src/tests/gamesRoutes.test.ts`
- `microservice-quizz/src/tests/aiEngineClient.test.ts`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Invalid quiz payload | ai-engine output violates expected quiz structure | generation fails or result is not stored | service tests, logs, validation errors | tighten prompt/validation and retry |
| Empty usable inventory | no valid rows or excessive duplicate rejection | random mobile flows degrade | random route checks, grouped summary | regenerate content or repair persisted records |
| AI auth circuit opens | repeated `401/403` or auth failures from ai-engine | service blocks new generation attempts | circuit snapshot and errors from `GenerationService` | fix auth and wait for cooldown or restart service |

## Security and access model

- quiz generation is private behind gateway and BFF layers
- ai-engine calls use internal API keys rather than public client credentials
- stored quiz content should not leak raw prompt or internal diagnostic context back to clients

## Deployment and rollback implications

### Deployment impact

- contract changes affect both upstream BFF consumers and ai-engine integration
- validator or canonicalization changes can alter duplicate detection and read compatibility

### Rollback impact

- rollback must consider already persisted rows generated under newer validation rules
- binary rollback alone does not clear bad generated content or open auth-circuit state

## Related capabilities

- [AI content generation](../ai/ai-content-generation.md)
- [Mobile gameplay orchestration](../player/mobile-gameplay-orchestration.md)
- [Seed-data preparation and repair](../platform/seed-data-preparation-and-repair.md)
- [Contract governance](../platform/contract-governance.md)

## Open questions

- how often persisted quiz rows should be revalidated after validator changes
- whether batch-generation and operator repair flows need stronger documented smoke coverage

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Primary validation workflows](../../validation/primary-workflows-validation.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
