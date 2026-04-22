# Key Sequence Flows

Last updated: 2026-04-22.

Sequence diagrams of the most important runtime flows. Each flow lists the use case it satisfies and the failure points to consider.

## SF-01 Mobile player starts a quiz round (UC-01)

```mermaid
sequenceDiagram
  autonumber
  participant App as Mobile App
  participant GW as api-gateway
  participant BFF as bff-mobile
  participant Q as microservice-quizz
  participant AI as ai-engine-api
  participant L as llama runtime

  App->>GW: GET /v1/games/quiz/random?...
  GW->>GW: Validate RandomGameQuerySchema
  GW->>BFF: forwardHttp(/quiz/random)
  BFF->>Q: GET /random?categoryId=...
  Q->>Q: select cached GameGeneration by difficulty (Prisma)
  alt cache hit
    Q-->>BFF: 200 generation payload
  else cache miss
    Q->>AI: POST /generate {category, difficulty}
    AI->>AI: optimizer + RAG retrieve + cacheKey
    AI->>L: POST /completion
    L-->>AI: model output
    AI-->>Q: 200 generation
    Q->>Q: persist GameGeneration (difficultyPercentage materialized)
    Q-->>BFF: 200 generation
  end
  BFF-->>GW: 200 normalized payload
  GW-->>App: 200
```

Failure points:
- Invalid query → 400 from gateway, never reaches BFF.
- AI cache miss + llama unreachable → 503 surfaced with correlation id.
- DB unreachable → 5xx, no partial writes.

## SF-02 Backoffice operator changes runtime AI target (UC-09)

```mermaid
sequenceDiagram
  autonumber
  participant SPA as backoffice
  participant GW as api-gateway
  participant BFF as bff-backoffice
  participant AI as ai-engine-api
  participant PV as mounted PV

  SPA->>GW: PATCH /admin/routing/ai-engine
  GW->>BFF: forward
  BFF->>BFF: authorize operator role
  BFF->>GW: PATCH /admin/runtime-target {url}
  GW->>PV: persist RoutingTarget(scope=gateway-ai)
  GW-->>BFF: 200
  opt also update llama
    BFF->>AI: PATCH /runtime/llama-target
    AI->>PV: persist RoutingTarget(scope=ai-llama)
    AI-->>BFF: 200
  end
  BFF-->>SPA: 200 effective target snapshot
```

Failure points:
- PV write failure → revert in-memory state, return 5xx.
- Drift vs manifests is expected; runbook documents reconciliation.

## SF-03 CI/CD build and staging deploy (UC-12, UC-13)

```mermaid
sequenceDiagram
  autonumber
  participant Dev as Developer
  participant SvcCI as Service CI
  participant Infra as platform-infra CI
  participant GHCR as GHCR
  participant K3s as k3s staging

  Dev->>SvcCI: push to main
  SvcCI->>SvcCI: lint + tests + audit (high)
  SvcCI->>Infra: repository_dispatch(build-push)
  Infra->>Infra: docker build (multi-stage, USER node)
  Infra->>GHCR: push :sha and :stg
  Infra->>Infra: deploy.yaml dispatch
  Infra->>K3s: kubectl apply -k overlays/stg
  K3s-->>Infra: rollout status
  Infra->>Infra: smoke checks
```

Failure points:
- Service CI failure → no dispatch.
- Image digest mismatch → deploy aborts.
- Smoke check fail → rollout marked unhealthy; manual intervention.

## SF-04 Operator inspects AI history with difficulty filter (UC-06)

```mermaid
sequenceDiagram
  autonumber
  participant SPA as backoffice
  participant GW as api-gateway
  participant BFF as bff-backoffice
  participant Q as microservice-quizz
  participant DB as quizz DB

  SPA->>GW: GET /admin/quiz/history?difficulty=hard&limit=...
  GW->>BFF: forward
  BFF->>BFF: cache lookup (TTL key normalized w/o querystring)
  alt cache hit
    BFF-->>SPA: 200 from cache
  else cache miss
    BFF->>Q: GET /historyPage?difficulty=...
    Q->>DB: SELECT ... WHERE difficultyPercentage BETWEEN ...
    DB-->>Q: rows
    Q-->>BFF: 200
    BFF->>BFF: store in metrics cache
    BFF-->>SPA: 200
  end
```

Failure points:
- DB slow query → BFF returns 504 after upstream timeout.
- Cache stampede → mitigated by short TTL + per-path key normalization.

## SF-05 AI ingest / corpus refresh (UC-11)

```mermaid
sequenceDiagram
  autonumber
  participant Op as Operator/Cron
  participant AI as ai-engine-api
  participant Vec as vector store
  participant Cache as Redis

  Op->>AI: POST /ingest {corpus_id}
  AI->>AI: load curated corpus
  AI->>AI: compute embeddings (MiniLM-L12)
  AI->>Vec: upsert vectors
  AI->>AI: compute new corpusSignature
  AI->>Cache: invalidate by signature prefix (logical)
  AI-->>Op: 202 ingestion accepted
```

Failure points:
- Embedding model load failure → ingestion aborts, previous signature retained.
- Vector store partial write → ingestion marked failed; retry idempotent.

## SF-06 Player submits gameplay result (UC-03)

```mermaid
sequenceDiagram
  autonumber
  participant App as Mobile App
  participant GW as api-gateway
  participant BFF as bff-mobile
  participant Q as microservice-quizz
  participant U as microservice-users

  App->>GW: POST /v1/games/quiz/result
  GW->>BFF: forward
  BFF->>Q: POST /rounds/:id/result
  Q->>Q: persist score
  Q-->>BFF: 200
  BFF->>U: POST /players/:id/events {gameResult}
  U->>U: update PlayerStats projection
  U-->>BFF: 202
  BFF-->>App: 200
```

Failure points:
- Q write fails → no analytics call; gateway returns 5xx.
- U analytics call fails → considered eventually consistent; logged for replay.

## Related documents

- [Use cases](./use-cases.md)
- [Domain model](./domain-model.md)
- [Target architecture](./target-architecture.md)
- [Runtime routing](../operations/runtime-routing-and-service-targeting.md)
