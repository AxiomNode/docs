# C4 Views

Last updated: 2026-11-14.

This document captures the AxiomNode platform using the C4 model: System Context (Level 1), Containers (Level 2), and Components (Level 3) for the most relevant containers. Level 4 (code) is intentionally omitted; it lives in each repository.

## Level 1 - System Context

```mermaid
C4Context
title AxiomNode - System Context

Person(player, "Mobile Player", "Plays quiz / wordpass games")
Person(operator, "Backoffice Operator", "Monitors platform and controls AI routing")
Person(platformEng, "Platform Engineer", "Builds, deploys and operates the platform")

System(axiom, "AxiomNode Platform", "Mobile + backoffice gaming platform with AI-assisted generation")

System_Ext(llama, "Llama Runtime", "Model server (in-cluster or external workstation)")
System_Ext(firebase, "Firebase", "Mobile identity and push (mobile-app)")
System_Ext(ghcr, "GitHub Container Registry", "Image storage")
System_Ext(github, "GitHub Actions", "CI / cross-repo dispatch")

Rel(player, axiom, "Plays games / consults stats", "HTTPS")
Rel(operator, axiom, "Operates and monitors", "HTTPS")
Rel(platformEng, github, "Pushes code, dispatches builds")
Rel(github, ghcr, "Publishes images")
Rel(github, axiom, "Manual deploy (workflow_dispatch; no staging host today)")
Rel(axiom, llama, "Generation requests", "HTTP")
Rel(player, firebase, "Auth / push", "HTTPS")
```

## Level 2 - Container View

```mermaid
C4Container
title AxiomNode - Containers

Person(player, "Mobile Player")
Person(operator, "Backoffice Operator")

System_Boundary(edge, "Edge") {
  Container(gateway, "api-gateway", "Node/Express", "Single public ingress, auth, CORS, routing; stateless (no routing state)")
}

System_Boundary(experience, "Experience / BFF") {
  Container(bffMobile, "bff-mobile", "Node", "Mobile-shaped contracts")
  Container(bffBack, "bff-backoffice", "Node", "Backoffice-shaped contracts + runtime control plane")
  Container(spa, "backoffice SPA", "React/Vite + nginx-unprivileged", "Operator UI")
}

System_Boundary(domain, "Domain") {
  ContainerDb(usersDb, "users DB", "PostgreSQL")
  Container(users, "microservice-users", "Node + Prisma", "Identity, profile, leaderboard, analytics")
  ContainerDb(quizDb, "quizz DB", "PostgreSQL")
  Container(quiz, "microservice-quizz", "Node + Prisma", "Quiz domain + AI request orchestration")
  ContainerDb(wpDb, "wordpass DB", "PostgreSQL")
  Container(wp, "microservice-wordpass", "Node + Prisma", "Wordpass domain + AI request orchestration")
}

System_Boundary(ai, "AI") {
  Container(aiApi, "ai-engine-api", "Python/FastAPI", "RAG + cache-aware generation, persists active llama target")
  Container(aiStats, "ai-engine-stats", "Python/FastAPI", "AI metrics and cache visibility")
  ContainerDb(redis, "ai cache", "Redis", "Cache + embedding store")
  Container_Ext(llama, "llama runtime", "C++ server", "External or in-cluster")
}

System_Boundary(platform, "Platform / Governance") {
  Container(platformInfra, "platform-infra", "kustomize + GH workflows", "Build, push, deploy")
  Container(contracts, "contracts-and-schemas", "OpenAPI + JSON Schema", "Hand-written docs validated by CI; no code generation")
  Container(sdk, "shared-sdk-client", "TypeScript", "Internal TypeScript library (shared schemas, HTTP proxy, game runtime)")
  Container(secretsRepo, "secrets", "GitOps repo", "Sealed material")
  Container(obs, "observability-platform", "Prometheus/Grafana stack", "Telemetry + alerts")
}

Rel(player, gateway, "HTTPS")
Rel(operator, spa, "HTTPS")
Rel(spa, gateway, "HTTPS")
Rel(gateway, bffMobile, "HTTP")
Rel(gateway, bffBack, "HTTP")
Rel(gateway, aiApi, "HTTP (/internal/ai-engine, upstream from env)")
Rel(bffMobile, quiz, "HTTP")
Rel(bffMobile, wp, "HTTP")
Rel(bffMobile, users, "HTTP")
Rel(bffBack, users, "HTTP")
Rel(bffBack, quiz, "HTTP")
Rel(bffBack, wp, "HTTP")
Rel(bffBack, aiStats, "HTTP")
Rel(bffBack, aiApi, "HTTP (llama target, plugin capabilities)")
Rel(quiz, gateway, "HTTP (generation, /internal/ai-engine)")
Rel(wp, gateway, "HTTP (generation, /internal/ai-engine)")
Rel(aiApi, llama, "HTTP")
Rel(aiApi, redis, "TCP")
Rel(aiStats, redis, "TCP")
Rel(users, usersDb, "TCP")
Rel(quiz, quizDb, "TCP")
Rel(wp, wpDb, "TCP")
Rel(platformInfra, gateway, "deploys")
Rel(sdk, gateway, "imports (also both BFFs, users, quizz, wordpass)")
Rel(sdk, bffMobile, "imports")
Rel(sdk, bffBack, "imports")
```

## Level 3 - Components

### api-gateway

- `RouteRegistry` — declarative route map, version-aware.
- `Forwarder` (`proxy` SDK module) — `forwardHttp` with structured upstream errors.
- `ContractGuards` — Zod-based validation for shared public query contracts.
- `EdgePolicy` — auth, CORS, rate-limit hooks.

### bff-backoffice

- `OperatorAuth` — session and role resolution against `microservice-users`.
- `LlamaTargetControl` — proxies llama target read/set/reset/probe to `ai-engine-api` `/internal/admin/llama-target`; keeps no routing state.
- `MetricsAggregator` — adapts upstream metrics caches with normalized TTL keys.
- `RowsQueryEngine` — UI-side filter with precomputed lowercase row strings.

### ai-engine-api

- `RAGRetriever` — multilingual MiniLM embeddings + metadata-aware oversample/rerank.
- `Optimizer` — query enrichment with category name, model, corpus signature.
- `CacheManager` — keyed by category, embedding model, corpus signature.
- `LlamaClient` — points to active llama target (persisted in `AI_ENGINE_LLAMA_TARGET_STATE_FILE`).
- `Diagnostics` — structured counters surfaced via FastAPI hooks.

### microservice-quizz / wordpass

- `Domain` — Prisma models incl. `GameGeneration` with `difficultyPercentage`.
- `AIClient` — calls `ai-engine-api` with category-aware payload.
- `HistoryQuery` — DB-level difficulty filter (no over-fetch).
- `RandomModelSelector` — same DB-level filter pattern.

### platform-infra

- `BuildPushWorkflow` — receives dispatch, builds, publishes `:sha`/`:stg`/`:prod`.
- `DeployWorkflow` — manual-only (`workflow_dispatch`): `kubectl apply -k overlays/{env}` with smoke checks; no staging host exists today.
- `Overlays` — `dev`, `stg`, `prod` (+ optional full-AI overlay).
- `SealedSecrets` — per-overlay sealed material.

## Notes on diagram authority

- These views describe the current platform state; aspirational changes go to ADRs first.
- Effective runtime topology can deviate from this diagram for the AI subsystem; see [target architecture](./target-architecture.md) and [runtime routing](../operations/runtime-routing-and-service-targeting.md).
