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
| `ai-engine` | 7000 (stats), 7001 (api), 7002 (llama) | AI generation, RAG, model serving, AI observability | Redis, llama server |

## Delivery-chain classification

### Automatic GHCR-to-staging chain

These repositories participate in the current automatic image publication and staging rollout chain:

- `api-gateway`
- `bff-mobile`
- `bff-backoffice`
- `backoffice`
- `microservice-users`
- `microservice-quizz`
- `microservice-wordpass`

### Outside the current automatic staging chain

- `mobile-app`: validated in its own CI, but not published as a k3s runtime image through `platform-infra`
- `ai-engine`: architecturally relevant, but currently optional in-cluster for staging because the active topology can externalize it to a workstation or relay target

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
- staging can therefore operate with `ai-engine` outside the cluster while the rest of the services remain auto-deployed on k3s

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

## Ownership model

- Service repos own service correctness, local tests, and channel/domain behavior.
- `platform-infra` owns packaging and deployment policy.
- `docs` owns cross-repository architecture and operations narratives.
- `contracts-and-schemas` owns contract source of truth.
- `shared-sdk-client` owns reusable client and validation primitives.
