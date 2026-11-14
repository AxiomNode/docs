# AI Observability And Diagnostics

Last updated: 2026-11-14.

## Purpose

This capability exposes the operational picture of AI behavior to operators by combining ai-engine stats, BFF aggregation, and backoffice diagnostics panels.

## Functional objective

- surface AI runtime health, cache state, and usage signals
- enrich raw AI stats with routing and operational context
- provide operator diagnostics screens that distinguish deployed topology from effective runtime behavior

## Ownership and boundaries

### Main owners

- `ai-engine-stats`
- `bff-backoffice`
- `backoffice`

### Explicit boundaries

- owns operator-facing AI diagnostics and aggregated AI visibility
- consumes generation/runtime signals from the AI content-generation capability
- reads the llama target state but does not own the runtime-target mutation path itself (`ai-engine-api` persists it)

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | ai-engine stats endpoints | aggregate AI runtime and event signals |
| HTTP route | BFF AI target, probe, and diagnostics routes | operator aggregation and enrichment path |
| UI flow | `AIDiagnosticsPanel.tsx` and related backoffice screens | interactive operator surface |

### Inputs

- AI event streams and stats queries emitted by ai-engine
- cache runtime data from ai-engine internal endpoints
- the effective llama target read back from `ai-engine-api` through the BFF
- operator requests initiated from the backoffice UI

### Outputs

- AI diagnostics views in backoffice
- enriched stats and runtime summaries through BFF routes
- operator-visible health, target, and probe results

### Contract sources

- `backoffice/src/src/ui/panels/AIDiagnosticsPanel.tsx`
- `backoffice/src/src/application/services/operationalSummary.ts`
- `bff-backoffice/src/app/routes/backoffice.ts`
- ai-engine internal stats and cache endpoints plus their tests

## Main functional flow

1. ai-engine records local events and forwards them to ai-stats on a best-effort basis.
2. ai-engine stats exposes aggregate runtime information for internal consumers.
3. `bff-backoffice` reads AI target, probe, and diagnostics data and enriches it with routing context.
4. `backoffice` fetches those summaries and renders diagnostic panels for operators.
5. Operators compare health, routing, and runtime evidence before taking action.

## Persistence and runtime state

### Owned persistence

- collector state in ai-stats
- none for routing: the BFF keeps no routing state (the llama target is persisted by `ai-engine-api`)

### Runtime state

- effective AI targets shape the meaning of every visible diagnostic
- ai-engine cache runtime and startup state influence what operators see
- UI panel state can lag behind live routing changes if upstream state changes between refreshes

### State risks

- stale collector or cache data making AI behavior appear healthier than it is
- UI, BFF, and effective runtime target disagreeing during or after target changes
- split AI topology causing one component to look healthy while the end-to-end flow is degraded

## Observability evidence

### Health signals

- ai-engine `/health` and `/ready`
- ai-engine stats availability
- BFF AI target and probe read routes

### Operational signals

- `AIDiagnosticsPanel.tsx`
- `fetchServiceOperationalSummary()` and related backoffice operational summary calls
- BFF AI target and probe endpoints

### Verification evidence

- `ai-engine/src/tests/test_observability/test_api.py`
- `bff-backoffice/src/tests/backoffice.test.ts`
- `bff-backoffice/src/tests/kubernetesObservability.test.ts`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Partial visibility | ai-stats degraded or internal cache endpoint unavailable | operators misread AI health | missing panels, BFF diagnostic errors | restore stats source or degrade clearly |
| Stale routing picture | BFF/UI state not refreshed after target mutation | wrong operational decision | compare target APIs with panel output | refresh state and re-probe |
| Split-topology confusion | API, stats, and llama target do not fail together | incident triage slows down | compare probe, health, and effective target data | isolate the failing segment and rerun probe |

## Security and access model

- AI diagnostics are intentionally operator-facing and should remain behind protected backoffice and BFF surfaces
- diagnostics can reveal internal hosts, ports, and routing state and therefore should not be exposed publicly
- event and cache data should avoid leaking prompts, tokens, or secret-bearing payloads

## Deployment and rollback implications

### Deployment impact

- stats, BFF, and UI changes must stay compatible across the diagnostics contract
- runtime target changes can alter the interpretation of diagnostics without any code deployment

### Rollback impact

- partial rollback can desynchronize what the UI expects from what the BFF or ai-stats exposes
- degraded diagnostics during rollback increase incident risk if not labeled clearly

## Related capabilities

- [AI content generation](./ai-content-generation.md)
- [Runtime routing control](../operations/runtime-routing-control.md)
- [Backoffice operational diagnostics](../operations/backoffice-operational-diagnostics.md)
- [Monitoring and alerting](../platform/monitoring-and-alerting.md)

## Open questions

- whether every operator-visible AI metric has a clearly documented owner and raw source
- whether degraded diagnostics are always labeled as partial instead of silently presenting stale data

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Backoffice operational diagnostics](../operations/backoffice-operational-diagnostics.md)
- [AI content generation](./ai-content-generation.md)
