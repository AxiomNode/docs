# Repository Map

Last updated: 2026-04-19.

## Runtime repositories

| Repository | Port(s) | Responsibility | Depends on |
|---|---|---|---|
| `api-gateway` | 7005 | Public edge ingress, auth, CORS, routing to BFFs | `bff-mobile`, `bff-backoffice` |
| `bff-mobile` | 7010 | Mobile-oriented orchestration for quiz/word-pass APIs | `microservice-quizz`, `microservice-wordpass` |
| `bff-backoffice` | 7011 | Admin-oriented orchestration (auth, monitoring, generation) | `microservice-users`, `microservice-quizz`, `microservice-wordpass`, `ai-engine` |
| `backoffice` | 7080 | React SPA for operations and admin workflows | `api-gateway` |
| `microservice-users` | 7102 (DB: 7434) | Identity, profile, leaderboard, gameplay analytics | PostgreSQL |
| `microservice-quizz` | 7100 (DB: 7432) | Quiz generation, persistence, and retrieval | PostgreSQL, `ai-engine` |
| `microservice-wordpass` | 7101 (DB: 7433) | Word-pass generation, persistence, and retrieval | PostgreSQL, `ai-engine` |
| `ai-engine` | 7000 (stats), 7001 (api), 7002 (llama) | AI generation, RAG, cache-aware generation, model serving, AI observability | Redis, llama server |

## Runtime state ownership

| Repository | Persisted runtime state | Why it matters |
|---|---|---|
| `api-gateway` | Active ai-engine API/stats target | Determines where live AI traffic is forwarded |
| `bff-backoffice` | Service target overrides and shared ai-engine presets | Shapes operator-visible effective topology |
| `ai-engine` | Active llama target inside `ai-engine-api` | Controls where generation calls reach the model runtime |

## Client and operator entry points

| Consumer | Entry point | Downstream path |
|---|---|---|
| Mobile app | `api-gateway` | `api-gateway` -> `bff-mobile` -> quiz/word-pass services |
| Backoffice SPA | `api-gateway` | `api-gateway` -> `bff-backoffice` -> users/services/ai diagnostics |
| Internal AI producers | service-local clients | quiz/word-pass -> `ai-engine-api` |
| Operators changing AI routing | `backoffice` through `bff-backoffice` | backoffice -> `bff-backoffice` -> `api-gateway` / `ai-engine-api` |

## Delivery-chain classification

### Automatic GHCR-to-staging chain

These repositories participate in the current automatic image publication and staging rollout chain:

- `api-gateway`
- `bff-mobile`
- `bff-backoffice`
- `backoffice`
- `ai-engine-api`
- `ai-engine-stats`
- `microservice-users`
- `microservice-quizz`
- `microservice-wordpass`

### Outside the current automatic staging chain

- `mobile-app`: validated in its own CI, but not published as a k3s runtime image through `platform-infra`
- external llama runtime: still operationally external in the split staging topology unless the optional full in-cluster AI overlay is rendered deliberately

## Platform repositories

| Repository | Responsibility |
|---|---|
| `contracts-and-schemas` | Canonical OpenAPI/JSON Schema/event contracts. |
| `shared-sdk-client` | Shared SDKs and integration utilities across services. |
| `platform-infra` | Build/push orchestration and Kubernetes deployment assets. |
| `observability-platform` | Monitoring stack, dashboards, and alert rules. |
| `docs` | Cross-repository architecture and operational documentation. |
| `secrets` | Centralized sensitive configuration governance and distribution. |

## Service communication flow

```text
Clients (mobile/backoffice)
  -> api-gateway (:7005)
     -> bff-mobile (:7010) -> microservice-quizz (:7100) / microservice-wordpass (:7101)
     -> bff-backoffice (:7011) -> microservice-users (:7102) / quizz / wordpass / ai-engine
```

## Effective runtime topology note

For AI-related flows, the effective runtime topology may differ from the deployed Kubernetes topology.

Reason:

- `api-gateway` can hold the active ai-engine target at runtime
- `bff-backoffice` can persist shared ai-engine presets and service-target overrides
- `ai-engine-api` can hold the active llama target at runtime
- staging can therefore operate in split-AI mode even while the rest of the services remain auto-deployed on k3s

## Current CI/CD automation map

- Service repos on `main` dispatch image builds to `platform-infra` (`build-push.yaml`).
- `platform-infra` deploy workflow runs after successful build and currently auto-deploys to `stg`.
- Staging and production promotion remain controlled and non-automatic in current policy.
- Dispatch now occurs only after the service repository validation jobs succeed.
- Docs-only pushes should not dispatch central image builds from the covered service repositories.

## Shared SDK modules (TypeScript)

| Module | Main consumers | Exports |
|---|---|---|
| `contracts` | BFF and service repositories | request/response contract schemas |
| `proxy` | `api-gateway`, BFF services | forwarding helpers and upstream error wrappers |
| `ai-engine-client` | quiz/wordpass services | AI engine client wrappers and config models |
| `trivia-categories` | quiz/wordpass services | language/category catalogs |
| `private-docs` | internal services | private docs token resolution and auth helpers |

## Documentation placement rule of thumb

- keep cross-repository topology, delivery rules, and runtime routing policy in `docs`
- keep repository-specific routes, ports, local workflows, and smoke checks in each repository README or local docs folder
- when a runtime state holder changes, update both the owning repository docs and the central runtime-routing documents

## Ownership model

- Service repos own service correctness, local tests, and channel/domain behavior.
- `platform-infra` owns packaging and deployment policy.
- `docs` owns cross-repository architecture and operations narratives.
- `contracts-and-schemas` owns contract source of truth.
- `shared-sdk-client` owns reusable client and validation primitives.
