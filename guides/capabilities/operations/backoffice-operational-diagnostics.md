# Backoffice Operational Diagnostics

Last updated: 2026-11-14.

## Purpose

This capability gives operators a single UI and BFF-backed surface to inspect service health, logs, metrics, the packaged backoffice version, and AI runtime context.

## Functional objective

- centralize operational diagnostics for the platform runtime
- aggregate heterogeneous downstream signals into operator-friendly views
- support safe runtime intervention by showing both symptoms and effective topology

## Ownership and boundaries

### Main owners

- `backoffice`
- `bff-backoffice`

### Downstream dependencies

- `microservice-users`
- `microservice-quizz`
- `microservice-wordpass`
- `ai-engine`
- `api-gateway`

### Explicit boundaries

- owns operator-facing visualization in backoffice panels such as `ServiceOverviewPanel.tsx` and `ServiceConsolePanel.tsx`
- owns BFF aggregation logic in `bff-backoffice/src/app/routes/backoffice.ts`
- exposes the packaged backoffice version, but does not own the health of the underlying services it displays
- overlaps with runtime routing control only where diagnostics must reflect current effective targets

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| UI flow | backoffice dashboards and operational panels | operator entry surface |
| HTTP route | BFF metrics, logs, and data routes | aggregated diagnostics data |
| Service call | backoffice `operationalSummary.ts` | front-end composition of diagnostics calls |

### Inputs

- operator auth/session state from backoffice
- downstream service metrics, logs, and data endpoints
- effective target state and AI probe results where relevant

### Outputs

- service-overview panels and drill-down console views
- enriched diagnostics summaries through the BFF
- operator-visible packaged version

### Contract sources

- `backoffice/src/src/application/services/operationalSummary.ts`
- `backoffice/src/src/ui/panels/ServiceOverviewPanel.tsx`
- `backoffice/src/src/ui/panels/ServiceConsolePanel.tsx`
- `bff-backoffice/src/app/routes/backoffice.ts`

## Main functional flow

1. An operator opens a diagnostics screen in backoffice.
2. The UI calls gateway-exposed backoffice routes that land in `bff-backoffice`.
3. The BFF queries downstream services.
4. The UI merges those responses with the packaged version into operator panels for health, logs, metrics, and the llama target.
5. Operators use the same surface to correlate a visible issue with the current llama target.

## Persistence and runtime state

### Owned persistence

- UI screen state only: the BFF keeps no routing state

### Runtime state

- downstream availability directly affects what panels can render
- operator screen state may lag behind new probes or target changes until refreshed

### State risks

- partial downstream outage makes the diagnostics picture incomplete or inconsistent
- missing proxy exposure at gateway level breaks whole panels despite healthy downstream code
- stale llama target context leads to wrong incident conclusions

## Observability evidence

### Health signals

- backoffice and BFF health

### Operational signals

- service metrics/logs endpoints
- packaged backoffice version display
- service overview and console panels in the UI

### Verification evidence

- `backoffice/src/src/test/service-overview-panel.integration.test.tsx`
- `backoffice/src/src/test/service-console-panel.integration.test.tsx`
- `bff-backoffice/src/tests/backoffice.test.ts`

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Missing proxy or route exposure | gateway or BFF route drift | diagnostics page fails even if the underlying service is healthy | UI error state, edge 404, route smoke checks | restore the route chain and revalidate |
| Partial downstream visibility | one downstream data source unavailable | operator sees incomplete runtime picture | missing panels and failed BFF calls | degrade clearly and restore the missing source |
| Stale llama target context | cached or unrefreshed panel state | wrong operational action | compare diagnostics output with direct target reads | refresh and reconcile state |

## Security and access model

- diagnostics surfaces are operator-facing and should remain protected by backoffice auth and route guards
- logs, metrics, and target state can reveal internal topology and must not leak publicly
- read-only diagnostics and mutating operational controls should remain clearly separated in the UI and BFF API

## Deployment and rollback implications

### Deployment impact

- UI and BFF changes need compatible rollout windows so panels do not call nonexistent routes
- gateway route exposure is a hidden dependency for operator visibility and must be included in rollout validation

### Rollback impact

- partial rollback can leave the UI expecting routes an older BFF or gateway version does not expose
- diagnostics panels remain useful only if rollback preserves minimum visibility paths

## Related capabilities

- [Runtime routing control](./runtime-routing-control.md)
- [AI observability and diagnostics](../ai/ai-observability-and-diagnostics.md)
- [Deployment and rollout automation](../platform/deployment-and-rollout-automation.md)
- [Monitoring and alerting](../platform/monitoring-and-alerting.md)

## Open questions

- whether every critical operator screen has explicit end-to-end smoke coverage through gateway, BFF, and downstream data sources
- whether degraded or partial data is always labeled clearly enough in the UI

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Backoffice React development](../../experience/backoffice-react-development.md)
- [Runtime routing control](./runtime-routing-control.md)
