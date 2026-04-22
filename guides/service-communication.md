# Service Communication Model

Last updated: 2026-04-05.

## Synchronous request flow

1. Mobile or backoffice client calls `api-gateway` (`:7005`).
2. `api-gateway` applies edge policies (token check, CORS, correlation ID, route constraints).
3. `api-gateway` forwards to the correct BFF using shared forwarding utilities.
4. BFF services orchestrate internal service calls and shape response payloads.

## Communication layers and allowed directions

| Source | Allowed target | Notes |
|---|---|---|
| Client applications | `api-gateway` | Public boundary only |
| `api-gateway` | `bff-mobile`, `bff-backoffice`, internal ai-engine proxy routes | No direct client access to domain services |
| `bff-mobile` | domain services | BFF-to-BFF calls are not allowed |
| `bff-backoffice` | domain services, ai diagnostics endpoints, gateway admin sync | Backoffice runtime control path is explicit |
| quiz/word-pass services | `ai-engine-api`, `ai-engine-stats` | Internal AI producer traffic |
| `ai-engine-api` | llama runtime | Model execution path, may be externalized |

## Public API surface (`api-gateway` -> BFFs)

### Mobile routes (through `bff-mobile`)

| Method | Route | Upstream target |
|---|---|---|
| GET | `/v1/mobile/games/quiz/random` | `microservice-quizz:/games/models/random` |
| GET | `/v1/mobile/games/wordpass/random` | `microservice-wordpass:/games/models/random` |
| POST | `/v1/mobile/games/quiz/generate` | `microservice-quizz:/games/generate` (generation timeout profile) |
| POST | `/v1/mobile/games/wordpass/generate` | `microservice-wordpass:/games/generate` (generation timeout profile) |

### Backoffice routes (through `bff-backoffice`)

| Method | Route | Upstream target |
|---|---|---|
| POST | `/v1/backoffice/auth/session` | `microservice-users:/users/firebase/session` |
| GET | `/v1/backoffice/auth/me` | `microservice-users:/users/me/profile` |
| GET | `/v1/backoffice/users/leaderboard` | `microservice-users:/users/leaderboard` |
| GET | `/v1/backoffice/monitor/stats` | `microservice-users:/monitor/stats` |
| GET | `/v1/backoffice/services/:service/metrics` | service-specific `/monitor/stats` |
| GET | `/v1/backoffice/services/:service/logs` | service-specific `/monitor/logs` |
| POST | `/v1/backoffice/services/:service/generation/process` | service generation process endpoint |

### Internal AI routes (through `api-gateway`)

| Method | Route | Upstream target |
|---|---|---|
| POST | `/internal/ai-engine/generate/quiz` | active `ai-engine-api` target |
| POST | `/internal/ai-engine/generate/word-pass` | active `ai-engine-api` target |
| POST | `/internal/ai-engine/ingest/quiz` | active `ai-engine-api` target |
| POST | `/internal/ai-engine/ingest/word-pass` | active `ai-engine-api` target |
| GET | `/internal/ai-engine/catalogs` | active `ai-engine-api` target |
| GET | `/internal/ai-engine/health` | active `ai-engine-api` target |
| GET | `/internal/ai-engine/stats` | active `ai-engine-stats` target |

### Runtime control routes

| Method | Route | Owner |
|---|---|---|
| GET/PUT/DELETE | `/internal/admin/ai-engine/target` | `api-gateway` |
| GET/PUT/DELETE | `/v1/backoffice/ai-engine/target` | `bff-backoffice` |
| GET/POST/PUT/DELETE | `/v1/backoffice/ai-engine/presets` | `bff-backoffice` |
| GET/PUT/DELETE | `/v1/backoffice/service-targets/:service` | `bff-backoffice` |

## Asynchronous flow

- Batch generation endpoints return `taskId` values.
- Clients poll process-status endpoints for progress/completion.
- Event contracts are versioned in `contracts-and-schemas/schemas/events/`.

## Effective AI request path

For generation-related traffic, the logical dependency chain is:

1. client calls gateway route
2. gateway or BFF forwards to the appropriate domain service
3. domain service calls `ai-engine-api`
4. `ai-engine-api` resolves the active llama target
5. llama runtime returns model output
6. `ai-engine-api` records stats and returns validated payloads upstream

This means an apparently healthy deploy can still fail functionally if the persisted live AI target or llama target points to an unreachable runtime.

## Communication rules

- Domain services are private and not directly internet-facing.
- BFF-to-BFF calls are disallowed.
- Timeouts and retry strategy must be explicit per dependency.
- `x-correlation-id` must be propagated end-to-end.
- Environment defaults and runtime overrides must be observable independently.
- Auth/security headers to preserve when applicable:
	- `authorization`
	- `x-firebase-id-token`
	- `x-dev-firebase-uid`
	- `x-api-key`

## Layered authentication model

| Layer | Mechanism |
|---|---|
| Client -> `api-gateway` | `Authorization: Bearer EDGE_API_TOKEN` |
| `bff-backoffice` -> `ai-engine` | `X-API-Key: AI_ENGINE_BRIDGE_API_KEY` |
| quiz/wordpass services -> `ai-engine` | `X-API-Key: AI_ENGINE_API_KEY` |
| Backoffice UI -> Firebase | Firebase ID token |
| `bff-backoffice` -> `microservice-users` | Forwarded Firebase token headers |

## Failure boundaries to document during incidents

- edge failure: gateway cannot reach BFF or rejects auth
- BFF orchestration failure: downstream service unavailable or timed out
- domain failure: validation, persistence, or bad stored content
- AI runtime failure: `ai-engine-api` or `ai-engine-stats` unhealthy
- model-runtime failure: llama target unreachable, overloaded, or warm-up incomplete
