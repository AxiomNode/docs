# AI Content Generation

Last updated: 2026-11-14.

## Purpose

This capability turns internal generation requests into validated game payloads, using ai-engine as the execution boundary and the game microservices as the main orchestration consumers.

## Functional objective

- generate structured quiz and word-pass payloads for downstream persistence and gameplay
- support RAG-backed ingestion and contextual generation inside ai-engine
- keep model execution isolated behind internal routes, API keys, and bridge services
- expose canonical catalogs and health signals used by game services and operational tooling

## Ownership and boundaries

### Main owners

- ai-engine generation API in `ai-engine/src/ai_engine/api/app.py`
- game-domain orchestration in `microservice-quizz/src/app/services/generationService.ts`
- equivalent word-pass orchestration in `microservice-wordpass/src/app/services/generationService.ts`

### Downstream dependencies

- llama runtime configured through ai-engine internal target state
- RAG pipeline and optimization/cache layer inside ai-engine
- ai-stats event sink for asynchronous observability forwarding
- Prisma-backed persistence in quiz and word-pass microservices

### Explicit boundaries

- owns generation request normalization, structured payload validation, ingest execution, and catalog publication inside ai-engine
- owns domain-side duplicate detection, storage, batch generation, and AI auth circuit control in the game microservices
- delegates actual model token generation to the llama runtime and monitoring aggregation to ai-stats
- does not expose direct public client access; callers enter through internal gateway routes or backoffice operational endpoints
- does not own public gameplay APIs, routing persistence, or rollout mechanics; those belong to edge/runtime control capabilities

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | `POST /internal/ai-engine/generate/quiz` in api-gateway | internal proxy to `POST /generate/quiz` in ai-engine |
| HTTP route | `POST /internal/ai-engine/generate/word-pass` in api-gateway | internal proxy to `POST /generate/word-pass` in ai-engine |
| HTTP route | `POST /internal/ai-engine/ingest/quiz` and `POST /internal/ai-engine/ingest/word-pass` in api-gateway | bridge-only ingest into ai-engine RAG |
| HTTP route | `GET /internal/ai-engine/catalogs` and `GET /internal/ai-engine/health` in api-gateway | catalog discovery and liveness for internal consumers |
| HTTP route | `GET/PUT/DELETE /v1/backoffice/ai-engine/target`, `POST /v1/backoffice/ai-engine/probe` in bff-backoffice | operator control for the llama target of the same runtime |
| Service method | `GenerationService.generateAndStore*()` in quiz and word-pass microservices | main domain-side consumer, dedup, persistence, batch orchestration |

### Inputs

- query text, item count, difficulty, optional category metadata, and cache flags
- `X-API-Key` with games or bridge scope depending on route family
- `X-Game-Language` and `X-Difficulty-Percentage` headers resolved by ai-engine generation routes
- ingest document batches with optional `X-Ingest-Source` override for telemetry
- operator-provided llama target payloads from backoffice for llama target changes and connectivity probes

### Outputs

- validated structured quiz, word-pass, and internally available true-false payloads
- ingest acknowledgements with document counts
- canonical category/language catalogs
- local generation events plus best-effort forwarded observability events to ai-stats
- persisted generated models in the game microservice databases after duplicate filtering

### Contract sources

- FastAPI request builders in `ai_engine.api.app.create_app()` and `_build_model_generate_request()`
- shared TypeScript client contract in `shared-sdk-client/typescript/src/ai-engine-client.ts`
- canonicalization/validation in shared SDK helpers such as `canonicalizeQuizModel` and `canonicalizeWordpassModel`
- runtime tests in `ai-engine/src/tests/test_api/test_app.py` and domain orchestration tests in the quiz and word-pass services

## Main functional flow

1. A domain service builds a request in `GenerationService.generateAndStoreWithResult()`, adding `query`, `item_count`, fixed `max_tokens`, and optional `difficulty_percentage`.
2. The shared `AiEngineClient` sends the request to the internal ai-engine base URL, applies retry/backoff, and caches catalog lookups for 60 seconds by default.
3. api-gateway forwards `/internal/ai-engine/*` to ai-engine API or ai-engine stats, resolving the upstreams from `AI_ENGINE_API_URL` and `AI_ENGINE_STATS_URL`.
4. ai-engine generation routes normalize inputs through query params plus `X-Game-Language` and `X-Difficulty-Percentage`, then call `_execute_generate()`.
5. `_execute_generate()` enforces rate and capacity limits, resolves context through the RAG pipeline when available, invokes the wrapped generator, validates the structured payload, records a local event, and forwards observability data to ai-stats on a best-effort basis.
6. The quiz and word-pass microservices canonicalize the response again, compute a content-based uniqueness key, reject duplicates, and persist only valid new models.
7. Ingest routes follow the same bridge path into `_execute_ingest()`, update the RAG pipeline, invalidate cached RAG diagnostics, and emit ingest telemetry with `ingest_model` and `ingest_source` metadata.

## Persistence and runtime state

### Owned persistence

- ai-engine local llama target override stored through `LlamaTargetStore`
- ai-engine cache and optimization runtime state exposed through `/internal/cache/stats`
- RAG-ingested content and derived diagnostics cache inside ai-engine
- persisted generated game rows in Prisma-backed `gameGeneration` tables owned by quiz and word-pass services

### Runtime state

- ai-engine startup state exposes `pending`, `initializing`, `ready`, or `failed` through `/health` and `/ready`
- generation capacity limiter tracks in-flight and queued work; health surfaces that dependency status explicitly
- microservice auth circuit state tracks failure streak, cooldown, open-until timestamp, and total openings
- catalogs in the microservices can come from `ai-engine` or fall back locally when remote lookup fails
- operators may repoint the llama target (held by `ai-engine-api`) without changing domain service code; the gateway and the BFF hold no routing state

### State risks

- drift between the persisted llama target override and the environment default can produce confusing partial failures after a redeploy
- stale category caches in consumers can hide upstream recovery or degradation for up to the configured TTL
- RAG diagnostics are cached and explicitly invalidated on ingest; broken invalidation would surface stale health data
- the auth circuit in domain services can intentionally block generation even after ai-engine recovers until cooldown expires

## Observability evidence

### Health signals

- `GET /health` and `GET /ready` in ai-engine expose startup state, dependency diagnostics, event count, and generation-capacity status
- `GET /internal/ai-engine/health` in api-gateway gives internal consumers a stable ingress point

### Operational signals

- ai-engine emits local collector events and forwards them to ai-stats via `_publish_event_to_stats()` when a stats URL is configured
- `GET /internal/ai-engine/stats` in api-gateway points to ai-engine stats rather than the main FastAPI app
- `GET /internal/cache/stats` and `POST /internal/cache/reset` expose cache runtime controls for monitoring bridges
- backoffice target and probe routes provide operator-level checks before switching runtime targets

### Verification evidence

- `ai-engine/src/tests/test_api/test_app.py` verifies generation-capacity health, quiz and word-pass model-specific routes, category-aware retrieval, ingest source override, and bridge-key-only ingest access
- quiz and word-pass `generationService.test.ts` verify remote catalog refresh, local fallback, auth circuit opening on `401`, duplicate detection, batch generation accounting, and ingest source precedence
- `shared-sdk-client/typescript/src/ai-engine-client.ts` encodes the retry, timeout, and catalog-cache behavior used by both game services

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Unreachable ai-engine API or llama target | bad llama target override, gateway upstream misconfiguration, runtime outage | generation fails or backoffice probes return invalid target errors | ai-engine `/health`, backoffice probe, gateway proxy failures, outbound request metrics | reset the target (`DELETE`), restore the runtime, or reapply a known-good target |
| Auth circuit opens in game services | repeated `401/403` or auth failures from ai-engine | quiz or word-pass generation is blocked before new upstream calls are attempted | `GenerationService` circuit snapshot, observer callbacks, test-covered error string `AI auth circuit open until ...` | fix API keys, wait for cooldown, or restart the affected microservice if operationally needed |
| Invalid structured output | model drift, prompt regression, or schema mismatch | payload is rejected or not stored downstream | canonicalization/validation failures, model-failed callbacks, history validation errors | adjust prompts or validators and retry with smoke checks |
| Stale diagnostics or catalogs | cache reuse without timely refresh | operators see outdated health/catalog state | RAG stats cache behavior, catalog timestamps, backoffice catalog view | force refresh via consumer call paths or invalidate cache through internal controls |
| Ingest accepted with wrong source context | missing or wrong `X-Ingest-Source` / query source | observability and corpus provenance become misleading | collector history metadata, ingest telemetry | resend ingest with explicit source and, if needed, refresh dependent corpora |

## Security and access model

- ai-engine keeps `/health` and `/ready` public for probes, but generation routes require a games or bridge key and ingest routes require the bridge key specifically
- backoffice target mutations are validated with Zod schemas in bff-backoffice before they can affect runtime routing
- api-gateway exposes ai-engine generation only on internal routes; the public edge does not call ai-engine directly
- prompt inputs, ingest documents, and generated payloads are normalized and canonicalized before persistence to reduce malformed-content and duplicate-content risks

## Deployment and rollback implications

### Deployment impact

- ai-engine API, ai-engine stats, and llama runtime are separable runtime units, so a healthy API deployment does not guarantee a healthy model target
- the llama target override (held by `ai-engine-api`) can redirect generation traffic without redeploying the game services
- changes to shared client retries, validation helpers, or prompt shaping affect both quiz and word-pass behavior immediately after rollout

### Rollback impact

- rolling back ai-engine code without resetting an operator-applied target override can preserve the same failing upstream target
- rolling back game microservices does not remove already persisted generated content or previously ingested RAG documents
- catalog fallback means a partial rollback can still appear superficially healthy while generation remains degraded

## Related capabilities

- [AI observability and diagnostics](./ai-observability-and-diagnostics.md)
- [Runtime routing control](../operations/runtime-routing-control.md)
- [Quiz inventory and generation](../domain/quiz-inventory-and-generation.md)
- [Word-pass inventory and generation](../domain/wordpass-inventory-and-generation.md)

## Open questions

- the quiz tests still assert effective language as English on model-specific generation paths even when the route defaults use `X-Game-Language=es`; that contract should be clarified explicitly at the platform level
- true-false generation and ingest routes exist in ai-engine, but no microservice, BFF route or mobile game uses them (checked on `origin/main`, 2026-11-12) and the gateway does not proxy them; the platform only plays quiz and word-pass
- the persisted llama target override outlives pod restarts and image rollbacks, so it is easy to forget during incident response; reset it explicitly

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [AI prompting rules](../../governance/ai-prompting-rules.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
