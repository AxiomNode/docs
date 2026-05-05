# Runtime Routing Control

Last updated: 2026-05-03.

## Purpose

This capability lets operators change effective service and AI routing at runtime, persist those overrides, and return safely to defaults without rebuilding the platform.

## Functional objective

- persist and apply runtime target overrides and presets
- expose the effective routing picture to operators
- support controlled rollback from overrides back to default topology

## Ownership and boundaries

### Main owners

- `bff-backoffice`
- `api-gateway`
- `ai-engine`

### Upstream actors

- backoffice operators

### Explicit boundaries

- owns BFF-side service target and AI preset management in `bff-backoffice/src/app/routes/backoffice.ts`
- owns gateway-side AI target persistence in `api-gateway/src/app/services/routingStateStore.ts`
- owns ai-engine local llama target override in `ai-engine`
- does not replace deploy-time configuration management in `platform-infra`, but can override runtime behavior after deploy

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | `GET/PUT/DELETE /v1/backoffice/ai-engine/target` | BFF AI target inspection and mutation |
| HTTP route | `GET/POST/PUT/DELETE /v1/backoffice/ai-engine/presets*` | shared AI preset management |
| HTTP route | `/v1/backoffice/service-targets/:service` | runtime service override control |
| HTTP route | `GET/PUT/DELETE /internal/admin/ai-engine/target` | gateway-owned AI target control |
| HTTP route | ai-engine internal llama target routes | ai-engine local runtime override |

### Inputs

- operator target changes and preset payloads validated by Zod schemas
- current runtime state from BFF routing store, gateway routing store, and ai-engine target store
- service identifiers such as `ai-engine-api`, `ai-engine-stats`, and other BFF-managed upstreams

### Outputs

- persisted override files and preset state
- effective runtime target changes
- reset-to-default side effects when overrides are cleared

### Contract sources

- `bff-backoffice/src/app/routes/backoffice.ts`
- `bff-backoffice/src/app/services/routingStateStore.ts`
- `api-gateway/src/app/services/routingStateStore.ts`
- `api-gateway/src/app/routes/proxy.ts`

## Main functional flow

1. An operator changes a target or preset in backoffice.
2. `bff-backoffice` validates the payload and persists the change in its routing-state store.
3. When the change affects AI execution, the BFF synchronizes runtime state toward `api-gateway` and ai-engine as needed.
4. Subsequent requests use the new effective target without a redeploy.
5. Operators verify health and can clear the override to fall back to defaults.

## Persistence and runtime state

### Owned persistence

- BFF service-target overrides and AI preset records
- gateway AI target override file
- ai-engine local llama target override file

### Runtime state

- effective topology is computed from environment defaults plus one or more persisted overrides
- overrides survive restart and redeploy unless explicitly cleared
- BFF, gateway, and ai-engine can each hold part of the routing truth

### State risks

- stale override after a healthy rollout leaves runtime using an unexpected target
- sync-order mistakes across BFF, gateway, and ai-engine create split routing state
- UI can display one topology while the actual traffic path uses another

## Observability evidence

### Health signals

- health probes executed after target changes

### Operational signals

- target and preset readback routes in the BFF and gateway
- operator probe route `POST /v1/backoffice/ai-engine/probe`
- runtime-target comparison in backoffice diagnostics

### Verification evidence

- `bff-backoffice/src/tests/routingStateStore.test.ts`
- `bff-backoffice/src/tests/backofficeRouteGuards.test.ts`
- `api-gateway/src/tests/routingStateStore.test.ts`
- `api-gateway/src/tests/proxy.test.ts`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Wrong target configured | operator error or stale preset | runtime requests fail while pods stay healthy | target read routes and smoke requests | reset to default or apply a known-good preset |
| State mismatch across owners | failed sync or concurrent edits | UI shows one topology while runtime uses another | compare BFF, gateway, and ai-engine target reads | resynchronize holders and retest |
| Override survives too long | runtime holder not cleared after rollout | traffic keeps pointing to a temporary host | post-deploy routing inspection | explicitly clear override and verify |

## Security and access model

- routing mutation paths are high-impact admin operations and should remain role-gated and auditable
- preset and target payloads can reveal sensitive internal topology and must stay on protected surfaces
- route guards in BFF and gateway are part of the control boundary, not optional UI behavior

## Deployment and rollback implications

### Deployment impact

- routing overrides can outlive container rollouts and therefore change the effective behavior of an otherwise healthy deployment
- gateway, BFF, and ai-engine changes must preserve routing-state compatibility across restarts

### Rollback impact

- rolling back binaries does not automatically roll back persisted routing state
- safe rollback often requires both image rollback and explicit override reset or reapplication

## Related capabilities

- [Public edge routing](../edge/public-edge-routing.md)
- [AI content generation](../ai/ai-content-generation.md)
- [AI observability and diagnostics](../ai/ai-observability-and-diagnostics.md)
- [Backoffice operational diagnostics](./backoffice-operational-diagnostics.md)

## Open questions

- whether all routing transitions are audited consistently enough across BFF, gateway, and ai-engine holders
- whether concurrent operator edits need stronger conflict semantics and UI signaling

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
- [Backoffice operational diagnostics](./backoffice-operational-diagnostics.md)
