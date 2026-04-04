# Local Edge Integration

## Goal

Run `api-gateway`, `bff-mobile`, and `bff-backoffice` in one composition for local integrated testing.

## Prerequisites

1. Domain microservices running on host:
   - `microservice-quizz` on `7100`
   - `microservice-wordpass` on `7101`
   - `microservice-users` on `7102`
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

## Shutdown

```bash
docker compose -f docker-compose.edge-integration.yml down
```
