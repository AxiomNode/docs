# Runtime Routing Control

Last updated: 2026-11-13.

## Purpose

This capability lets operators point `ai-engine-api` at a different llama runtime without a redeploy, persist that override, and return to the environment default. It is the only runtime routing control that remains; the full flow is in [Runtime routing and service targeting](../../../operations/runtime-routing-and-service-targeting.md).

## Ownership and boundaries

| Owner | Responsibility | Code |
|---|---|---|
| `Backoffice` | llama target card (`AiEngineTargetControl`) in AI diagnostics | `Backoffice/src/src/ui/components/AiEngineTargetControl.tsx` |
| `api-gateway` | plain forwarding of `/v1/backoffice/*` to the BFF | `api-gateway/src/app/routes/routeTable.ts` |
| `bff-backoffice` | validation, host allowlist, probe, sync to ai-engine | `bff-backoffice/src/app/routes/backoffice.ts` |
| `ai-engine-api` | owns and persists the override | `ai-engine/src/ai_engine/api/app.py` |

Out of scope: deploy-time configuration (`platform-infra`) and any BFF-side routing state (none exists).

## Entrypoints and contracts

| Type | Route | Notes |
|---|---|---|
| HTTP | `GET/PUT/DELETE /v1/backoffice/ai-engine/target` | read, apply, clear |
| HTTP | `POST /v1/backoffice/ai-engine/probe` | non-mutating reachability check of `<target>/v1/models` |
| HTTP | `GET /v1/backoffice/ai-engine/plugin/capabilities` | effective plugin runtime capabilities |
| HTTP (private) | `GET/PUT/DELETE /internal/admin/llama-target` on `ai-engine-api` | called by the BFF with `x-api-key` |

Payload for `PUT` and `POST probe`: `host`, `protocol` (`http` or `https`), `port`, optional `label`, validated with Zod in the BFF.

## Main functional flow

1. The operator edits the llama target card and optionally probes it.
2. The BFF checks the host against `ALLOWED_ROUTING_TARGET_HOSTS` and calls `ai-engine-api`.
3. `ai-engine-api` persists the override in `AI_ENGINE_LLAMA_TARGET_STATE_FILE` and applies it.
4. The BFF clears its upstream cache and breakers and returns the read-back target.
5. `DELETE` restores the environment default.

## Persistence and risks

- State lives only in `ai-engine-api` (file on mounted storage); it survives restarts and redeploys.
- A stale override after a rollout leaves generation pointing at an unexpected runtime while pods look healthy.
- Rolling back images does not roll back the override.

## Failure modes

| Failure mode | Likely cause | Detection | Recovery |
|---|---|---|---|
| `400` on `PUT` | host not allowed or invalid payload | response message | fix the payload or `ALLOWED_ROUTING_TARGET_HOSTS` |
| Probe not reachable | wrong host/port or runtime down | probe result | correct the target |
| Target set but generation fails | llama runtime unreachable or warming up | smoke request, `ai-engine-api` health | `DELETE` the override or fix the runtime |
| Admin call times out | `ai-engine-api` hung | BFF error citing the timeout (`AI_ENGINE_ADMIN_TIMEOUT_MS`) | restore `ai-engine-api` |

## Security and access model

- Mutating routes are high-impact admin operations behind the backoffice authentication path.
- Targets are restricted by `ALLOWED_ROUTING_TARGET_HOSTS` (denied outside development when empty).
- `ai-engine-api` internal admin routes are not public.

## Observability and verification

- Readback via `GET /v1/backoffice/ai-engine/target`; reachability via the probe route.
- Tests: `bff-backoffice/src/tests/backoffice.test.ts`, `backofficeBranches.test.ts`, `routingAllowlist.test.ts`; `ai-engine/src/tests/test_api/test_llama_target_store.py`; `Backoffice/src/src/test/ai-diagnostics-panel.integration.test.tsx`.

## History

Service-target overrides, shared presets and connections, the Kubernetes overview, routing history, and the matching Backoffice tab and presets selector were removed (`bff-backoffice` PR #9, `Backoffice` PR #7). The matching dead forwarding entries and the gateway's own `/internal/admin/ai-engine/target` store were removed from `api-gateway` (PR #8).

## Related capabilities

- [Public edge routing](../edge/public-edge-routing.md)
- [AI content generation](../ai/ai-content-generation.md)
- [AI observability and diagnostics](../ai/ai-observability-and-diagnostics.md)
- [Backoffice operational diagnostics](./backoffice-operational-diagnostics.md)

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
