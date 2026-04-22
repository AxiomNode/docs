# C4 Views

Last updated: 2026-04-22.

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
Rel(github, axiom, "Deploys to k3s")
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
  Container(gateway, "api-gateway", "Node/Express", "Single public ingress, auth, CORS, routing, persists active ai-engine target")
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
  Container(contracts, "contracts-and-schemas", "OpenAPI + JSON Schema", "Source of truth")
  Container(sdk, "shared-sdk-client", "TS/Python/Kotlin", "Generated clients")
  Container(secretsRepo, "secrets", "GitOps repo", "Sealed material")
  Container(obs, "observability-platform", "Prometheus/Grafana stack", "Telemetry + alerts")
}

Rel(player, gateway, "HTTPS")
Rel(operator, spa, "HTTPS")
Rel(spa, gateway, "HTTPS")
Rel(gateway, bffMobile, "HTTP")
Rel(gateway, bffBack, "HTTP")
Rel(gateway, aiApi, "HTTP (live target)")
Rel(bffMobile, quiz, "HTTP")
Rel(bffMobile, wp, "HTTP")
Rel(bffMobile, users, "HTTP")
Rel(bffBack, users, "HTTP")
Rel(bffBack, quiz, "HTTP")
Rel(bffBack, wp, "HTTP")
Rel(bffBack, aiStats, "HTTP")
Rel(bffBack, gateway, "HTTP (control plane mutations)")
Rel(quiz, aiApi, "HTTP (generation)")
Rel(wp, aiApi, "HTTP (generation)")
Rel(aiApi, llama, "HTTP")
Rel(aiApi, redis, "TCP")
Rel(aiStats, redis, "TCP")
Rel(users, usersDb, "TCP")
Rel(quiz, quizDb, "TCP")
Rel(wp, wpDb, "TCP")
Rel(platformInfra, gateway, "deploys")
Rel(contracts, sdk, "feeds")
Rel(sdk, gateway, "imports")
Rel(sdk, bffMobile, "imports")
Rel(sdk, bffBack, "imports")
```

## Level 3 - Components

### api-gateway

- `RouteRegistry` — declarative route map, version-aware.
- `Forwarder` (`proxy` SDK module) — `forwardHttp` with structured upstream errors.
- `ContractGuards` — Zod-based validation for shared public query contracts.
- `RuntimeTargetStore` — persists active ai-engine API/stats target on mounted PV.
- `EdgePolicy` — auth, CORS, rate-limit hooks.

### bff-backoffice

- `OperatorAuth` — session and role resolution against `microservice-users`.
- `ServiceTargetOverrideStore` — persists per-route upstream overrides.
- `AIPresetStore` — reusable ai-engine destination presets.
- `MetricsAggregator` — adapts upstream metrics caches with normalized TTL keys.
- `RowsQueryEngine` — UI-side filter with precomputed lowercase row strings.

### ai-engine-api

- `RAGRetriever` — multilingual MiniLM embeddings + metadata-aware oversample/rerank.
- `Optimizer` — query enrichment with category name, model, corpus signature.
- `CacheManager` — keyed by category, embedding model, corpus signature.
- `LlamaClient` — points to active llama target (persisted).
- `Diagnostics` — structured counters surfaced via FastAPI hooks.

### microservice-quizz / wordpass

- `Domain` — Prisma models incl. `GameGeneration` with `difficultyPercentage`.
- `AIClient` — calls `ai-engine-api` with category-aware payload.
- `HistoryQuery` — DB-level difficulty filter (no over-fetch).
- `RandomModelSelector` — same DB-level filter pattern.

### platform-infra

- `BuildPushWorkflow` — receives dispatch, builds, publishes `:sha`/`:stg`/`:prod`.
- `DeployWorkflow` — `kubectl apply -k overlays/{env}` with smoke checks.
- `Overlays` — `dev`, `stg`, `prod` (+ optional full-AI overlay).
- `SealedSecrets` — per-overlay sealed material.

## Notes on diagram authority

- These views describe the current platform state; aspirational changes go to ADRs first.
- Effective runtime topology can deviate from this diagram for the AI subsystem; see [target architecture](./target-architecture.md) and [runtime routing](../operations/runtime-routing-and-service-targeting.md).
