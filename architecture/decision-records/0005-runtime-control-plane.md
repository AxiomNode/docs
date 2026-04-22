# ADR 0005 - Backoffice-driven runtime control plane

- Status: Accepted
- Date: 2026-04-22

## Context

We need to change AI upstream routing without redeploying the platform (e.g., point `api-gateway` to a different `ai-engine-api`, or `ai-engine-api` to a different llama target). GitOps alone forces a build/deploy cycle that is too slow for AI recovery scenarios.

## Decision

Introduce a narrow runtime control plane operated through the backoffice:

- `bff-backoffice` persists service-target overrides and shared ai-engine presets.
- `api-gateway` persists the active ai-engine API/stats target on a mounted PV.
- `ai-engine-api` persists the active llama target on a mounted PV.

Mutations are operator-authenticated and survive pod restarts. The control plane does not replace declarative manifests; it complements them for AI routing only.

## Alternatives considered

1. Pod env vars + redeploy: rejected; too slow for recovery.
2. ConfigMap edits + rollouts: rejected; still requires platform-infra round-trip and lacks operator authentication.
3. Full service mesh with traffic shifting: rejected; out of scope for current scale.

## Consequences

- Two state planes (declared vs effective). Drift must be observable.
- Routing changes are auditable through structured logs but not through git history.
- Failure modes (PV write failure, partial mutation) are documented in runbooks.

## Related

- ADR 0004
- [Runtime routing operations doc](../../operations/runtime-routing-and-service-targeting.md)
- [QA-O1 in quality attributes](../quality-attributes.md)
