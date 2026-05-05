# Word-pass Inventory And Generation

Last updated: 2026-05-03.

## Purpose

This capability owns word-pass generation orchestration, quality validation, persistence, retrieval, and post-generation repair workflows used by player and operator channels.

## Functional objective

- generate coherent word-pass models through ai-engine
- preserve reusable inventory for mobile gameplay
- support maintenance and repair when generated hints or answers are not good enough

## Ownership and boundaries

### Main owners

- `microservice-wordpass`

### Downstream dependencies

- PostgreSQL via Prisma
- `ai-engine-api`

### Explicit boundaries

- owns word-pass route registration and storage semantics in `microservice-wordpass`
- owns duplicate detection and canonicalization via `canonicalizeWordpassModel()`
- consumes ai-engine through the shared client and shared orchestration helpers
- does not own mobile presentation, backoffice UI, or the actual LLM runtime

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | word-pass generation routes in `games.ts` | AI-assisted word-pass creation |
| HTTP route | `/games/models/random` | random word-pass retrieval |
| HTTP route | maintenance and history routes | operator-facing inspection and curation |
| Service method | `GenerationService.generateAndStoreWithResult()` | core generation and dedup path |

### Inputs

- generation requests with category, difficulty, optional letters, and item count
- inventory filters and operator maintenance queries
- ai-engine catalog and generation responses
- repair inputs applied through backoffice or tooling paths

### Outputs

- validated word-pass payloads persisted into inventory/history tables
- random word-pass inventory responses
- process snapshots and maintenance-facing history views
- repaired records through privileged operational workflows

### Contract sources

- `microservice-wordpass/src/app/routes/games.ts`
- `microservice-wordpass/src/app/services/generationService.ts`
- shared ai-engine client contract in `shared-sdk-client`
- shared validators and canonicalization helpers for word-pass models

## Main functional flow

1. A BFF or operator-facing route requests generation or inventory retrieval.
2. The word-pass service validates category, difficulty, count, and optional letters.
3. `GenerationService` calls ai-engine and canonicalizes the result.
4. The service derives a uniqueness key from normalized words/hints and checks for duplicates.
5. Valid new content is stored in Prisma and later exposed through random/history reads.
6. When quality is insufficient, operator repair flows update persisted content without re-running the full generation path.

## Persistence and runtime state

### Owned persistence

- word-pass inventory and history rows in PostgreSQL

### Runtime state

- cached category catalog source and AI auth circuit state
- in-memory generation-process tracking similar to the quiz service
- repair workflows can mutate persisted content after initial creation

### State risks

- repaired content can diverge from original generation metadata
- stale or low-quality hints can survive across releases if not rechecked
- readers can surface partially corrected rows when data operations are mid-flight

## Observability evidence

### Health signals

- word-pass service health and readiness

### Operational signals

- ai-engine outbound request metrics via the shared observer hook
- history and grouped-summary routes
- repair-script output and backoffice maintenance responses

### Verification evidence

- `microservice-wordpass/src/tests/generationService.test.ts`
- `microservice-wordpass/src/tests/gamesRoutes.test.ts`
- private-doc and service-metrics tests covering AI access and instrumentation behavior

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Bad hints or answers | weak generation or stale imported content | unplayable word-pass experience | validation tests, gameplay checks, repair queues | repair records and strengthen validation |
| Stale inventory | failed updates or partial repair | inconsistent player and operator views | random/history reads and backoffice inspection | rerun repair or regenerate affected records |
| AI auth circuit opens | repeated upstream auth failures | new generation is blocked | circuit snapshot and generation errors | fix auth path and retry after cooldown |

## Security and access model

- generation and maintenance routes are intended for protected internal callers only
- repair flows use privileged backoffice or tooling access and should be treated as admin operations
- stored content should not expose prompt internals or maintenance-only metadata to mobile consumers

## Deployment and rollback implications

### Deployment impact

- validator or schema changes can affect both newly generated content and stored repair compatibility
- operational repair scripts must track current route and payload expectations

### Rollback impact

- binary rollback does not undo repaired records or imported bad data
- safe rollback may require data restoration in addition to code rollback

## Related capabilities

- [AI content generation](../ai/ai-content-generation.md)
- [Mobile gameplay orchestration](../player/mobile-gameplay-orchestration.md)
- [Seed-data preparation and repair](../platform/seed-data-preparation-and-repair.md)
- [Contract governance](../platform/contract-governance.md)

## Open questions

- whether hint-quality and definition-quality policy is documented strongly enough outside service tests and scripts
- whether repair tooling should expose a more formal dry-run and audit trail model

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Primary validation workflows](../../validation/primary-workflows-validation.md)
- [Seed-data preparation and repair](../platform/seed-data-preparation-and-repair.md)
