# ADR 0001 - Single public edge via api-gateway

- Status: Accepted
- Date: 2026-04-22 (back-dated; reflects existing decision)

## Context

The platform exposes mobile and backoffice channels backed by multiple internal microservices and an AI subsystem. We needed a stable boundary for auth, CORS, rate-limiting, contract validation, and AI routing.

## Decision

Run a single public ingress, `api-gateway`, as the only externally reachable HTTP service. All client and operator traffic enters through it. Internal services (`bff-*`, `microservice-*`, `ai-engine-*`) are private to the cluster network.

## Alternatives considered

1. Direct client-to-BFF exposure: rejected; multiplies the public attack surface and pushes auth concerns into every BFF.
2. API gateway provided by an ingress controller only (no application gateway): rejected; we needed application-level concerns (contract validation, AI runtime target persistence).
3. GraphQL federation gateway: rejected; out of scope for current channels and adds operational complexity.

## Consequences

- Single choke point for edge policy.
- `api-gateway` becomes a routing-state holder (live AI target).
- Operational dependency: gateway downtime equals platform downtime; mitigated by k3s replicas and probes.

## Related

- [Repository map](../repository-map.md)
- [QA-A1, QA-S1 in quality attributes](../quality-attributes.md)
