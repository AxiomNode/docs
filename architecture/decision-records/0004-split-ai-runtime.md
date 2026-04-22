# ADR 0004 - Split AI runtime (services in-cluster, llama optional)

- Status: Accepted
- Date: 2026-04-22

## Context

The AI subsystem includes Python services (`ai-engine-api`, `ai-engine-stats`) and a llama runtime. Running llama in-cluster on staging hardware is expensive and sometimes unstable; running it externally on a workstation can be more practical for diagnostics and cost.

## Decision

Treat AI topology as a runtime mode matrix:

1. Split-AI: `ai-engine-api` and `ai-engine-stats` in-cluster, llama external.
2. Full in-cluster AI: optional overlay in `platform-infra` for controlled diagnostics.
3. Fully external AI: workstation-hosted llama reachable via runtime routing during recovery.

The active llama target is persisted by `ai-engine-api` on a mounted PV and modifiable via the runtime control plane.

## Alternatives considered

1. Always in-cluster llama: rejected; cost and stability concerns on current staging hardware.
2. Always external llama: rejected; loses the option of self-contained diagnostics.

## Consequences

- Effective topology can diverge from declared topology; documented as first-class concept.
- Runbook required to switch modes safely (`ai-services-recovery-runbook.md`).
- Cache key includes `corpus_signature` so corpus refreshes invalidate stale cache entries logically.

## Related

- [Target architecture](../target-architecture.md)
- [Runtime routing](../../operations/runtime-routing-and-service-targeting.md)
- ADR 0005
