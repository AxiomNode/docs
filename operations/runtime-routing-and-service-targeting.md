# Runtime Routing And Service Targeting

Last updated: 2026-11-13.

## Purpose

This document explains the one runtime routing control that exists outside the static Kubernetes manifests: the `ai-engine-api` llama target. Everything else is wired declaratively through `platform-infra` and environment variables.

## When to use this

- when diagnosing generation traffic that does not match the deployed Kubernetes manifests
- when an operator needs to apply, probe, inspect, or clear the llama target
- when validating how `Backoffice`, `bff-backoffice`, and `ai-engine-api` interact for that target

## What exists

The llama target is the host, port, and protocol that `ai-engine-api` uses to reach the model runtime (llama-compatible server). It defaults to the environment value and can be overridden at runtime, for example to point staging at a workstation-hosted model while API and stats stay in-cluster (see [ADR 0004](../architecture/decision-records/0004-split-ai-runtime.md)).

### Operator flow

1. The operator edits the llama target card in the `Backoffice` AI diagnostics view.
2. `Backoffice` calls `api-gateway`, which forwards `/v1/backoffice/ai-engine/*` to `bff-backoffice` as a plain proxy route.
3. `bff-backoffice` validates the payload, checks the host against `ALLOWED_ROUTING_TARGET_HOSTS`, and calls `ai-engine-api` at `/internal/admin/llama-target` with the service API key (`x-api-key`).
4. `ai-engine-api` rebuilds its llama URL, persists the override, and applies it to its runtime client.
5. The BFF clears its upstream cache and breaker state and returns the effective target read back from `ai-engine-api`.

### Routes

| Layer | Method and route | Notes |
|---|---|---|
| BFF (public through the gateway) | `GET /v1/backoffice/ai-engine/target` | effective target read back from `ai-engine-api` |
| BFF | `PUT /v1/backoffice/ai-engine/target` | body `host`, `protocol` (`http` or `https`, default `http`), `port` (default `7002`), optional `label` |
| BFF | `DELETE /v1/backoffice/ai-engine/target` | clear the override and return to the environment default |
| BFF | `POST /v1/backoffice/ai-engine/probe` | same body as `PUT`; checks `<target>/v1/models` without changing state |
| BFF | `GET /v1/backoffice/ai-engine/plugin/capabilities` | reads `ai-engine-api` `/internal/plugin/capabilities`; `502` when unreachable |
| `ai-engine-api` (private) | `GET/PUT/DELETE /internal/admin/llama-target` | owner of the override; not exposed publicly |
| `ai-engine-api` (private) | `GET /internal/plugin/capabilities` | effective plugin runtime capabilities |

## Persistence

- `ai-engine-api` stores the override in `AI_ENGINE_LLAMA_TARGET_STATE_FILE` (default `data/llama-target-state.json`), so it survives pod recreation. `DELETE` resets the file and falls back to the environment URL.
- `bff-backoffice` keeps no routing state: it has no routing-state store on `main`.

## Security boundaries

- `PUT` is rejected unless the host matches `ALLOWED_ROUTING_TARGET_HOSTS` (exact host, `*.suffix`, or IPv4 CIDR). When the variable is empty, overrides are denied outside development.
- The probe route performs a server-side request to an operator-supplied host; keep it on the authenticated backoffice surface.

## Gateway state (current, cleanup pending)

`api-gateway` still has a second, separate target store on `main`: `GET/PUT/DELETE /internal/admin/ai-engine/target` (guarded by `API_GATEWAY_ADMIN_TOKEN`, persisted in `GATEWAY_ROUTING_STATE_FILE`) that can redirect the `/internal/ai-engine/*` proxy. `bff-backoffice` no longer calls it, so the Backoffice flow above does not touch it. The gateway route table also still forwards a few paths that `bff-backoffice` no longer serves (`/kubernetes/overview`, `/ai-engine/presets*`, `/ai-engine/connections*`, `/service-targets*`); those requests end in an upstream error. No pull request that removes them is open in `AxiomNode/api-gateway` as of this date, so treat this as current behavior and an open cleanup item.

## Validation

- `GET /v1/backoffice/ai-engine/target` shows the effective target after a change or reset.
- `POST /v1/backoffice/ai-engine/probe` checks reachability before applying.
- A generation smoke request confirms end-to-end traffic reaches the intended runtime.

## Risks and cautions

- the persisted override outlives pod restarts and redeploys
- rolling back images does not clear the override; reset it explicitly
- operators reading only manifests will miss the effective llama target

## History

The BFF used to also expose service-target overrides (`/service-targets*`), shared ai-engine presets and connections (`/ai-engine/presets*`, `/ai-engine/connections*`), a Kubernetes overview (`/kubernetes/overview`), and routing history (`/routing/history`), with the Backoffice Kubernetes tab and presets selector. These were removed in `bff-backoffice` PR #9 and `Backoffice` PR #7; only the llama target remains.

## Related documents

- [Runtime routing control](../guides/capabilities/operations/runtime-routing-control.md)
- [Deployment strategy](./deployment-strategy.md)
- [Target architecture](../architecture/target-architecture.md)
- [CI/CD workflow map](./cicd-workflow-map.md)
