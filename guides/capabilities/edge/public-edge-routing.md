# Public Edge Routing

Last updated: 2026-05-03.

## Purpose

This capability provides the stable ingress boundary for mobile and backoffice clients while keeping downstream services private, replaceable, and operationally reroutable.

## Functional objective

- accept public traffic through one edge service
- apply edge auth, CORS, timeout, and forwarding policy before reaching BFFs
- expose internal AI proxy routes without making ai-engine public
- preserve a stable public surface even when runtime targets change underneath

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

- owns public ingress, route registration, forwarding, and AI proxy target resolution in `api-gateway/src/app/routes/proxy.ts`
- owns persisted AI target overrides in `api-gateway/src/app/services/routingStateStore.ts`
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
| HTTP route | `/internal/admin/ai-engine/target` | privileged runtime target read/update/delete |

### Inputs

- public HTTP requests from mobile and backoffice clients
- edge auth tokens and correlation metadata
- environment defaults for BFF and AI upstream URLs
- persisted AI routing override loaded from `GATEWAY_ROUTING_STATE_FILE`

### Outputs

- forwarded public responses for mobile and backoffice clients
- forwarded internal responses for ai-engine generation, ingest, health, and stats
- persisted side effects when operators mutate the AI target through admin routes

### Contract sources

- route registration and target logic in `api-gateway/src/app/routes/proxy.ts`
- edge bootstrap and request middleware in `api-gateway/src/app/server.ts`
- shared forwarding helpers such as `buildUrl()` and `forwardRequest()` from `shared-sdk-client`
- regression coverage in `api-gateway/src/tests/proxy.test.ts`

## Main functional flow

1. A public client reaches `api-gateway` on a `/v1/mobile/*` or `/v1/backoffice/*` route.
2. The gateway applies edge auth and request policy in `server.ts`, including health-route exemptions and route classification for rate limiting.
3. `proxy.ts` resolves the correct upstream target, using persisted AI overrides for `/internal/ai-engine/*` and static BFF URLs for public BFF paths.
4. The gateway forwards the request through shared HTTP helpers and route-specific circuit breakers.
5. The response is returned to the caller while preserving edge-visible status and correlation context.

## Persistence and runtime state

### Owned persistence

- gateway routing state persisted by `RoutingStateStore`

### Runtime state

- default upstream URLs from `api-gateway/src/app/config.ts`
- effective AI API and stats targets computed from env defaults plus persisted overrides
- per-upstream circuit-breaker state inside the gateway process

### State risks

- stale AI target after restart or redeploy because the routing state file survives
- gateway target drift relative to BFF or ai-engine runtime state
- public route availability diverging from operator expectations when the gateway lags behind downstream route additions

## Observability evidence

### Health signals

- `GET /health` and `GET /v1/health`

### Operational signals

- gateway logs and upstream timeout/failure traces
- admin readback from `/internal/admin/ai-engine/target`
- route-aware rate-limiter classification in `api-gateway/src/app/services/rateLimiter.ts`

### Verification evidence

- `api-gateway/src/tests/proxy.test.ts`
- `api-gateway/src/tests/routingStateStore.test.ts`
- focused health-route tests in `api-gateway/src/tests/health.test.ts`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Forwarding failure | BFF unreachable, timeout, or broken proxy route | client-visible outage or edge 404/5xx | proxy tests, logs, smoke requests | restore upstream or add/fix the missing proxy route |
| Wrong AI target | stale persisted override or bad admin update | internal AI routes fail while public gateway stays healthy | `/internal/admin/ai-engine/target`, AI smoke checks | clear or replace the persisted override |
| Partial route exposure | downstream route exists but gateway route is missing | feature appears broken from public edge only | end-to-end probe through gateway | add the corresponding gateway proxy route and retest |

## Security and access model

- public edge access is separated from privileged admin routing control
- `/internal/admin/ai-engine/target` is protected by the gateway admin token path, not public client auth
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
