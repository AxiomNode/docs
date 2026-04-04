# Service Communication Model

Last updated: 2026-04-05.

## Synchronous request flow

1. Mobile or backoffice client calls `api-gateway` (`:7005`).
2. `api-gateway` applies edge policies (token check, CORS, correlation ID, route constraints).
3. `api-gateway` forwards to the correct BFF using shared forwarding utilities.
4. BFF services orchestrate internal service calls and shape response payloads.

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

## Asynchronous flow

- Batch generation endpoints return `taskId` values.
- Clients poll process-status endpoints for progress/completion.
- Event contracts are versioned in `contracts-and-schemas/schemas/events/`.

## Communication rules

- Domain services are private and not directly internet-facing.
- BFF-to-BFF calls are disallowed.
- Timeouts and retry strategy must be explicit per dependency.
- `x-correlation-id` must be propagated end-to-end.
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
