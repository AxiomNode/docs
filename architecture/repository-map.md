# Repository Map

Ultima actualizacion: 2026-03-29.

## Repos de runtime

| Repo | Puerto | Responsabilidad | Depende de |
|------|--------|-----------------|------------|
| `api-gateway` | 7005 | Entrada publica, CORS, auth edge, proxy a BFFs | bff-mobile, bff-backoffice |
| `bff-mobile` | 7010 | Composicion para app mobile (quiz/wordpass random y generate) | microservice-quizz, microservice-wordpass |
| `bff-backoffice` | 7011 | Composicion para panel administrativo (auth, monitoring, catalogs, generation, roles) | microservice-users, microservice-quizz, microservice-wordpass, ai-engine |
| `backoffice` | 7080 | SPA React (Vite + TailwindCSS) servida por Nginx | api-gateway |
| `microservice-users` | 7102 (DB: 7434) | Identidad Firebase, perfil, estadisticas, roles, leaderboard | PostgreSQL |
| `microservice-quizz` | 7100 (DB: 7432) | Generacion, almacenamiento y distribucion de quizzes | PostgreSQL, ai-engine |
| `microservice-wordpass` | 7101 (DB: 7433) | Generacion, almacenamiento y distribucion de wordpass | PostgreSQL, ai-engine |
| `ai-engine` | 7000 (stats), 7001 (api), 7002 (llama) | Inferencia LLM, RAG, catalogos, metricas AI | Redis, Llama server |

## Repos de plataforma

| Repo | Responsabilidad |
|------|-----------------|
| `contracts-and-schemas` | OpenAPI, JSON Schema y eventos versionados. Fuente canonica de contratos. |
| `shared-sdk-client` | SDKs multi-lenguaje derivados de contratos. TypeScript publicado, Python/Kotlin placeholder. |
| `platform-infra` | IaC (Terraform/K8s) y docker-compose de integracion por entorno (dev/stg/prod). |
| `observability-platform` | Stack de metricas (Prometheus), logs, traces y alertas P0. |
| `docs` | Arquitectura, guias operativas y documentacion de referencia. |
| `secrets` | Catalogo centralizado de variables sensibles con scripts de bootstrap, inyeccion y auditoria. |

## Flujo de comunicacion

```
Cliente (mobile/backoffice)
  → api-gateway (:7005)
    → bff-mobile (:7010) → microservice-quizz (:7100) / microservice-wordpass (:7101)
    → bff-backoffice (:7011) → microservice-users (:7102) / quizz / wordpass / ai-engine
```

## Shared SDK Client — Modulos exportados

| Modulo | Consumido por | Exporta |
|--------|---------------|---------|
| `contracts` | bff-mobile, bff-backoffice, microservice-users | RandomGameQuerySchema, GenerateGameRequestSchema, LeaderboardQuerySchema |
| `proxy` | api-gateway, bff-mobile, bff-backoffice | forwardHttp, buildUrl, UpstreamTimeoutError, extractForwardHeaders |
| `ai-engine-client` | microservice-quizz, microservice-wordpass | AiEngineClient, interfaces de config y observabilidad |
| `trivia-categories` | microservice-quizz, microservice-wordpass | TRIVIA_CATEGORIES (24), SUPPORTED_LANGUAGES (5) |
| `private-docs` | microservice-quizz, microservice-users | resolvePrivateDocsToken, isAuthorizedForPrivateDocs |
