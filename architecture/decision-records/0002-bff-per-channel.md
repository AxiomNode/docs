# ADR 0002 - BFF per channel

- Status: Accepted
- Date: 2026-04-22

## Context

Mobile and backoffice clients have different latency budgets, payload shapes, and operator workflows. Sharing a single API surface for both produced friction in contracts and bloated payloads.

## Decision

Maintain one Backend-for-Frontend per channel:

- `bff-mobile` for the mobile app.
- `bff-backoffice` for the operator SPA.

Each BFF owns channel-specific shaping, caching, and orchestration of internal calls.

## Alternatives considered

1. Single shared BFF: rejected; couples mobile and operator evolution.
2. Direct client calls to multiple microservices: rejected; pushes orchestration into clients and increases coupling to internal topology.

## Consequences

- Two BFFs to maintain; partially mitigated by shared SDK modules.
- `bff-backoffice` evolved into a runtime control-plane host (see ADR 0005).
- Mobile contracts can change independently from operator-facing changes.

## Related

- [Repository map](../repository-map.md)
- [Use cases](../use-cases.md)
