# Repository Map

Last updated: 2026-04-05.

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

## Current CI/CD automation map

- Service repos on `main` dispatch image builds to `platform-infra` (`build-push.yaml`).
- `platform-infra` deploy workflow runs after successful build and currently auto-deploys to `stg`.
- Staging and production promotion remain controlled and non-automatic in current policy.

## Shared SDK modules (TypeScript)

| Module | Main consumers | Exports |
|---|---|---|
| `contracts` | BFF and service repositories | request/response contract schemas |
| `proxy` | `api-gateway`, BFF services | forwarding helpers and upstream error wrappers |
| `ai-engine-client` | quiz/wordpass services | AI engine client wrappers and config models |
| `trivia-categories` | quiz/wordpass services | language/category catalogs |
| `private-docs` | internal services | private docs token resolution and auth helpers |
