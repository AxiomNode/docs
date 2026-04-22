# Inter-Service Communication

Last updated: 2026-04-23.

Executive summary of the communication contracts between AxiomNode services.
The detailed operational source (routes, ports, headers) remains in
[`service-communication.md`](./service-communication.md). This document
captures the **hard rules** that any new service or route must respect.

## Allowed topology

| Source | Allowed target | Forbidden |
|---|---|---|
| Client (mobile, backoffice) | `api-gateway` (`:7005`) | Direct access to BFFs or services |
| `api-gateway` | `bff-mobile`, `bff-backoffice`, internal routes to `ai-engine` | Direct access to `microservice-*` |
| `bff-mobile` | Domain services (`microservice-quizz`, `microservice-wordpass`, `microservice-users`) | Calling another BFF |
| `bff-backoffice` | Domain services + `ai-engine` (admin/diagnostics) | Calling another BFF |
| `microservice-quizz`, `microservice-wordpass` | `ai-engine` (`api`, `stats`) | Cross-calls between domain services |
| `ai-engine-api` | llama runtime | Direct access from external clients |

## Synchrony and semantics

- **Sync by default**, over HTTP/JSON. BFFs may orchestrate multiple internal
  calls but must return a single cohesive response.
- **Async/jobs**: only for long tasks (training, batch eval). In that case
  the BFF responds with 202 + `job_id` and exposes `/jobs/{id}` for polling.
  No websockets between services.
- **Idempotency**: every mutation exposed in `api-gateway` must accept the
  `Idempotency-Key` header and honor it for at least 24 h.
- **Timeouts**: the BFF enforces a hard timeout of 8 s for domain calls and
  20 s for `ai-engine` (model) calls. Anything beyond degrades with the
  fallback documented in the contract.

## Mandatory headers

| Header | Source | Purpose |
|---|---|---|
| `X-Correlation-ID` | injected by `api-gateway` if missing | End-to-end traceability |
| `Authorization: Bearer <token>` | client / BFF | Public or internal authentication |
| `X-Internal-Token: <INTERNAL_API_TOKEN>` | `api-gateway` → BFF | Trusted-origin validation |
| `X-Distribution: dev|stg|pro` | `api-gateway` | Route and policy selection |

Token details: see
[`docs/operations/environments-and-secrets.md`](../operations/environments-and-secrets.md).

## Errors and response contract

- Uniform format `{ "error": { "code": string, "message": string, "details"?: object } }`.
- BFFs translate internal errors to business codes. They never leak stack
  traces, internal IDs or downstream service names.
- `api-gateway` adds `X-Correlation-ID` to error responses as well.

## Minimum observability per hop

Every service exposes:

- `/healthz` (cheap liveness, no dependencies).
- `/readyz` (readiness, validates critical dependencies).
- `/metrics` Prometheus with counter `<svc>_requests_total{route,status}` and
  histogram `<svc>_request_duration_seconds`.

For `ai-engine` the LLMOps metrics defined in ADR 0009 also apply, as
described in
[`ai-prompting-rules.md`](./ai-prompting-rules.md#mandatory-telemetry).

## Contract changes

1. Edit the OpenAPI of the service (`contracts-and-schemas/` when applicable
   or the service's own repo).
2. Validate with `validate-contracts` in CI.
3. Version the route (`/v1/...`, `/v2/...`); never break an active `/v1`.
4. Announce the change in the weekly retro
   ([`docs/operations/retrospective-process.md`](../operations/retrospective-process.md)).

## References

- [`service-communication.md`](./service-communication.md) — detailed contract.
- [`docs/operations/environments-and-secrets.md`](../operations/environments-and-secrets.md).
- [`docs/operations/secrets-rotation-policy.md`](../operations/secrets-rotation-policy.md).
- ADR 0006 (centralized platform-infra).
- ADR 0007 (immutable tags).
