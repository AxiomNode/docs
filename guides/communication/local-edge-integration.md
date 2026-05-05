# Local Edge Integration

Last updated: 2026-05-03.

## Purpose

Run `api-gateway`, `bff-mobile`, and `bff-backoffice` in one composition for local integrated testing.

## Scope

This guide is intentionally operational and local-only. It does not redefine the public edge contract or cross-repo routing ownership.

## Prerequisites

1. Domain microservices running on host:
   - `microservice-quizz` on `7100`
   - `microservice-wordpass` on `7101`
   - `microservice-users` on `7102`
   - `ai-engine-api` on `7001`
2. Docker Desktop running.
3. Dev secrets available in private `secrets` repository.
4. Run `node scripts/prepare-runtime-secrets.mjs dev` from `secrets`.

## Start edge layer

From `platform-infra/environments/dev`:

```bash
docker compose -f docker-compose.edge-integration.yml up -d --build
```

## Quick checks

```bash
curl http://localhost:7005/health
curl "http://localhost:7005/v1/mobile/games/quiz/random?language=es"
curl "http://localhost:7005/v1/backoffice/users/leaderboard?limit=5"
```

If `EDGE_API_TOKEN` is configured in the gateway, include:

```bash
-H "Authorization: Bearer <EDGE_API_TOKEN>"
```

`EDGE_API_TOKEN` is loaded from `api-gateway/src/.env.secrets`, copied from `secrets/runtime/repositories/api-gateway/dev.env`.

## Automated smoke test

```bash
bash scripts/smoke-edge.sh
```

The smoke now covers:

- gateway health
- unauthenticated mobile profile rejection
- invalid mobile profile payload rejection
- authenticated mobile profile read/update/read round-trip with persisted `playerId`, `displayName`, `preferredLanguage`, timestamp presence, and zero-state stats semantics
- deferred gameplay event sync with converged profile stats and duplicate-batch deduplication
- direct quiz AI generation through `microservice-quizz:/games/generate` and `microservice-quizz:/games/generate/process/wait`, followed by observable history growth and inventory selection through `/games/models/random`
- runtime service-target override for `microservice-quiz`, followed by an affected catalogs request and rollback to `env`
- dedicated `ai-engine target` override/readback/reset through backoffice routing controls
- mobile quiz and word-pass discovery routes
- selected mobile generate routes with invalid-payload rejection and playable response-envelope checks
- selected backoffice leaderboard and monitoring routes

The smoke intentionally does not pin optional identity fields such as `email` and `photoUrl`, because those are resolved from `microservice-users` and are better protected by repository-local tests than by an edge smoke that should stay resilient.

## Shutdown

```bash
docker compose -f docker-compose.edge-integration.yml down
```

## Related documents

- [Public edge routing](../capabilities/edge/public-edge-routing.md)
- [Runtime routing and service targeting](../../operations/runtime-routing-and-service-targeting.md)
