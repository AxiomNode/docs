# Use Cases

Last updated: 2026-04-22.

This document enumerates the primary use cases supported by the AxiomNode platform, the actors that trigger them, and the runtime path that satisfies them. It is intentionally implementation-aware so it can be cross-checked against contracts, BFFs, and microservices.

## Actors

| Actor | Channel | Authentication | Notes |
|---|---|---|---|
| Mobile Player | Mobile App (Android) | Firebase identity / opaque session via gateway | Plays games, consults stats and leaderboards |
| Backoffice Operator | Backoffice SPA | Operator credentials via `bff-backoffice` | Manages users, monitors AI generation, controls runtime routing |
| Internal Service | service-to-service (cluster network) | Internal trust boundary | quiz/wordpass call `ai-engine-api`; BFFs call domain services |
| Platform Engineer | CI / kubectl / runbooks | GitHub identity / k3s kubeconfig | Triggers builds, applies manifests, runs recovery runbooks |
| AI Engine | autonomous (cron / on-demand) | n/a | Generation, RAG retrieval, cache eviction |

## Use case catalogue

### UC-01 Play a quiz round (Mobile Player)

- Trigger: Player requests a new game from the mobile app.
- Path: `Mobile App -> api-gateway -> bff-mobile -> microservice-quizz`.
- Pre-conditions: Player session valid; categories catalog reachable.
- Post-conditions: A `GameGeneration` row exists with serialized prompt + difficulty; client receives a normalized game payload.
- Failure modes: domain service unavailable (gateway returns 5xx with correlation id); AI cache miss timeout (quiz returns deterministic fallback or error per service policy).

### UC-02 Play a word-pass round (Mobile Player)

- Trigger: Player requests a wordpass game.
- Path: `Mobile App -> api-gateway -> bff-mobile -> microservice-wordpass`.
- Equivalent to UC-01 with the wordpass domain.

### UC-03 Submit gameplay result (Mobile Player)

- Trigger: Player completes a round.
- Path: `Mobile App -> api-gateway -> bff-mobile -> microservice-{quizz|wordpass}` and `microservice-users` for analytics.
- Post-conditions: Score persisted; user analytics updated; leaderboard projection eventually consistent.

### UC-04 Consume leaderboard / stats (Mobile Player or Operator)

- Trigger: Player or operator opens stats screen.
- Path (mobile): `api-gateway -> bff-mobile -> microservice-users`.
- Path (operator): `api-gateway -> bff-backoffice -> microservice-users`.

### UC-05 Operator authentication (Backoffice Operator)

- Trigger: Operator logs into backoffice SPA.
- Path: `backoffice -> api-gateway -> bff-backoffice -> microservice-users`.
- Post-conditions: Session token issued; SPA stores per-route persisted state.

### UC-06 Inspect AI generation history (Backoffice Operator)

- Trigger: Operator opens the AI history screen.
- Path: `backoffice -> api-gateway -> bff-backoffice -> microservice-{quizz|wordpass}` for `historyPage` (Prisma-filtered by difficulty).
- Notes: difficulty filtering is performed at the database layer (see audit 2026-04-19).

### UC-07 Inspect AI engine stats and cache (Backoffice Operator)

- Trigger: Operator opens AI dashboards.
- Path: `backoffice -> api-gateway -> bff-backoffice -> ai-engine-stats`.
- Post-conditions: Operator sees per-cache-key metrics, embedding model in use, RAG corpus signature.

### UC-08 Trigger AI generation manually (Backoffice Operator)

- Trigger: Operator launches a controlled generation job.
- Path: `backoffice -> bff-backoffice -> api-gateway -> ai-engine-api`.
- Pre-conditions: Active llama target reachable from `ai-engine-api`.
- Post-conditions: Generation request enqueued; cache key and metrics updated.

### UC-09 Change runtime AI target (Backoffice Operator)

- Trigger: Operator selects a new ai-engine-api/stats target or llama upstream from the backoffice control plane.
- Path: `backoffice -> bff-backoffice -> api-gateway` (persists active target on PV) and/or `bff-backoffice -> ai-engine-api` (persists active llama target).
- Post-conditions: Effective routing diverges from declarative manifests; persisted on mounted volume; survives pod recreation.
- Risk: state drift versus manifests; must be observable via stats endpoints and runbook.

### UC-10 Generate quiz/wordpass via internal service (Internal Service)

- Trigger: `microservice-quizz` or `microservice-wordpass` requires AI generation.
- Path: `microservice-{quizz|wordpass} -> ai-engine-api -> llama runtime`.
- Notes: cache key includes `category_id`, embedding model, curated corpus signature.

### UC-11 Ingest curated corpus / refresh embeddings (AI Engine)

- Trigger: scheduled or operator-initiated ingest.
- Path: internal `ai-engine` ingestion pipeline.
- Post-conditions: Vector index refreshed; corpus signature changes; downstream caches invalidated by signature inclusion.

### UC-12 Build and publish a service image (Platform Engineer / CI)

- Trigger: push to `main` of a runtime service repo.
- Path: service CI validates, then dispatches to `platform-infra/build-push.yaml` which builds, pushes to GHCR (`:sha`, `:stg`), and triggers deploy.
- Post-conditions: Staging deployment updated to immutable SHA; `:stg` floating tag updated.

### UC-13 Deploy to staging / production (Platform Engineer)

- Trigger: dispatch from `build-push.yaml` (stg auto) or manual `publish_prod_tag` (prod).
- Path: `platform-infra/deploy.yaml -> kubectl apply -k overlays/{stg|prod}`.
- Pre-conditions: prod requires controlled dispatch; sealed-secrets present.

### UC-14 Validate contracts and SDKs (Platform Engineer / CI)

- Trigger: PR to `contracts-and-schemas` or `shared-sdk-client`.
- Path: `validate-contracts` / `typescript-sdk-ci` workflows.
- Post-conditions: Failing contract changes block downstream services from compiling against drift.

### UC-15 Recover AI services (Platform Engineer)

- Trigger: AI runtime degraded (llama down, embedding mismatch, cache poisoned).
- Path: follow `operations/ai-services-recovery-runbook.md`; may include UC-09 to repoint to workstation-hosted llama.
- Post-conditions: Effective topology recovered; documented in runbook log.

### UC-16 Distribute secrets (Platform Engineer)

- Trigger: secret rotation or new service onboarding.
- Path: edit `secrets` repo -> sealed-secrets in `platform-infra/overlays/*/sealed-secrets/` -> apply.

## Use case to repository matrix

| Use case | Edge | Experience | Domain | AI | Platform |
|---|---|---|---|---|---|
| UC-01..UC-04 | api-gateway | bff-mobile | quizz/wordpass/users | (cache-aware) | - |
| UC-05..UC-09 | api-gateway | bff-backoffice / backoffice | users | ai-engine-api/stats | - |
| UC-10..UC-11 | - | - | quizz/wordpass | ai-engine | - |
| UC-12..UC-16 | - | - | - | - | platform-infra / secrets / contracts / sdk |

## Out-of-scope use cases (explicitly excluded)

- Real-time multiplayer.
- Direct client-to-domain calls (must go through edge + BFF).
- Client-driven AI calls (mobile clients never call `ai-engine-*` directly).
- Manual pod edits as a routing mechanism (must use UC-09 control plane).

## Related documents

- [Repository map](./repository-map.md)
- [Target architecture](./target-architecture.md)
- [Key sequence flows](./key-sequence-flows.md)
- [Domain model](./domain-model.md)
- [Quality attributes](./quality-attributes.md)
