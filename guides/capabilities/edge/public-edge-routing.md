# Public Edge Routing

Last updated: 2026-11-13.

## Purpose

This capability provides the stable ingress boundary for mobile and backoffice clients while keeping downstream services private and replaceable.

## Functional objective

- accept public traffic through one edge service
- apply edge auth, CORS, timeout, and forwarding policy before reaching BFFs
- expose internal AI proxy routes without making ai-engine public
- preserve a stable public surface when downstream targets change

## Ownership and boundaries

### Main owners

- `api-gateway`

### Upstream actors

- `mobile-app`
- `backoffice`

### Downstream dependencies

- `bff-mobile`
- `bff-backoffice`
- `ai-engine-api`
- `ai-engine-stats`

### Explicit boundaries

- owns public ingress and the route table in `api-gateway/src/app/routes/routeTable.ts`, with forwarding and AI proxy target resolution in `proxy.ts`
- resolves the `/internal/ai-engine/*` upstreams only from `AI_ENGINE_API_URL` and `AI_ENGINE_STATS_URL`; it holds no persisted routing state (the runtime llama target lives in `ai-engine-api`, see [Runtime routing and service targeting](../../../operations/runtime-routing-and-service-targeting.md))
- delegates business orchestration to BFFs and AI execution to ai-engine
- does not own domain validation, persistence, or operator UI semantics

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | `GET /health` and `GET /v1/health` | cheap liveness surface in `api-gateway/src/app/routes/health.ts` |
| HTTP route | `/v1/mobile/*` | public ingress toward `bff-mobile` |
| HTTP route | `/v1/backoffice/*` | public ingress toward `bff-backoffice` |
| HTTP route | `/internal/ai-engine/*` | internal proxy for generation, ingest, catalogs, health, and stats |

### Inputs

- public HTTP requests from mobile and backoffice clients
- edge auth tokens and correlation metadata
- environment defaults for BFF and AI upstream URLs

### Outputs

- forwarded public responses for mobile and backoffice clients
- forwarded internal responses for ai-engine generation, ingest, health, and stats

### Contract sources

- route registration and target logic in `api-gateway/src/app/routes/proxy.ts`
- edge bootstrap and request middleware in `api-gateway/src/app/server.ts`
- shared forwarding helpers such as `buildUrl()` and `forwardRequest()` from `shared-sdk-client`
- regression coverage in `api-gateway/src/tests/proxy.test.ts`

## Main functional flow

1. A public client reaches `api-gateway` on a `/v1/mobile/*` or `/v1/backoffice/*` route.
2. The gateway applies edge auth and request policy in `server.ts`, including health-route exemptions and route classification for rate limiting.
3. `proxy.ts` resolves the upstream from the route table, using the environment-configured AI targets for `/internal/ai-engine/*` and static BFF URLs for public BFF paths.
4. The gateway forwards the request through shared HTTP helpers and route-specific circuit breakers.
5. The response is returned to the caller while preserving edge-visible status and correlation context.

## Persistence and runtime state

### Owned persistence

- none

### Runtime state

- upstream URLs from `api-gateway/src/app/config.ts` (BFF URLs, `AI_ENGINE_API_URL`, `AI_ENGINE_STATS_URL`)
- per-upstream circuit-breaker state inside the gateway process

### State risks

- gateway upstream URLs drifting from the real BFF or ai-engine addresses after a redeploy
- public route availability diverging from operator expectations when the gateway lags behind downstream route additions or removals (the route table is the single list to update)

## Observability evidence

### Health signals

- `GET /health` and `GET /v1/health`

### Operational signals

- gateway logs and upstream timeout/failure traces
- route-aware rate-limiter classification in `api-gateway/src/app/services/rateLimiter.ts`

### Verification evidence

- `api-gateway/src/tests/proxy.test.ts`
- `api-gateway/src/tests/proxyRouteEquivalence.test.ts`
- focused health-route tests in `api-gateway/src/tests/health.test.ts`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Forwarding failure | BFF unreachable, timeout, or broken proxy route | client-visible outage or edge 404/5xx | proxy tests, logs, smoke requests | restore upstream or add/fix the missing proxy route |
| Wrong AI target | `AI_ENGINE_API_URL` or `AI_ENGINE_STATS_URL` points at the wrong host | internal AI routes fail while public gateway stays healthy | AI smoke checks | fix the env value and redeploy |
| Partial route exposure | downstream route exists but gateway route is missing | feature appears broken from public edge only | end-to-end probe through gateway | add the corresponding gateway proxy route and retest |

## Security and access model

- the gateway exposes no admin routing control; the runtime llama target is changed through the authenticated Backoffice surface and lives in `ai-engine-api`
- internal AI proxy routes stay private to the platform and are not directly exposed on public client flows

## Deployment and rollback implications

### Deployment impact

- gateway deployment can change public behavior independently of BFF or ai-engine deploys
- AI behavior at the edge depends on both deployed gateway code and persisted target state

### Rollback impact

- binary rollback does not clear the persisted AI target override
- safe rollback requires revalidating effective targets, not just container image version

## Related capabilities

- [Mobile gameplay orchestration](../player/mobile-gameplay-orchestration.md)
- [Backoffice operational diagnostics](../operations/backoffice-operational-diagnostics.md)
- [Runtime routing control](../operations/runtime-routing-control.md)
- [AI content generation](../ai/ai-content-generation.md)

## Open questions

- whether every newly added BFF route has a matching gateway smoke check before rollout
- whether route-level auth and tracing guarantees are documented consistently enough for future edge additions

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Service communication model](../../communication/service-communication.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
