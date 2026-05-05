# AI Services Recovery Runbook (Dev)

Last updated: 2026-05-03.

## Purpose

Quickly recover `ai-engine-api` and `ai-engine-stats` in the local dev runtime when they appear down or fail with authentication errors.

## When to use this

- local `ai-engine-api` or `ai-engine-stats` health checks return non-`200`
- backoffice AI metrics or logs endpoints fail while the local AI stack should be available
- `401` or `403` responses appear between `bff-backoffice`, `api-gateway`, and the local AI services

## Preconditions

- local Docker runtime available
- working copy of the `ai-engine` repository
- access to the effective dev keys used by `bff-backoffice`, `api-gateway`, and `ai-engine`

## Execution steps

### Startup and health

1. Go to `ai-engine/src`.
2. Run `docker compose up -d ai-cache ai-stats ai-api`.
3. Verify:
   - `GET http://localhost:7001/health` returns `200`
   - `GET http://localhost:7000/health` returns `200`

### Minimum operational smoke

1. Obtain the effective runtime keys for `ai-stats`:
   - `AI_ENGINE_API_KEY`
   - `AI_ENGINE_BRIDGE_API_KEY`
2. Validate monitoring endpoints:
   - `GET http://localhost:7000/stats` with `X-API-Key` returns `200`
   - `GET http://localhost:7000/stats/history?last_n=5` with `X-API-Key` returns `200`
3. Validate aggregated consumption through the edge:
   - `GET /v1/backoffice/services/ai-engine-api/metrics` returns `200`
   - `GET /v1/backoffice/services/ai-engine-api/logs?limit=5` returns `200`
   - `GET /v1/backoffice/services/ai-engine-stats/metrics` returns `200`
   - `GET /v1/backoffice/services/ai-engine-stats/logs?limit=5` returns `200`

### Key recovery if `401` or `403` appears

1. Check for key mismatch between `bff-backoffice` and the `ai-engine` runtime.
2. Restore the standard dev keys in `platform-infra/environments/dev/docker-compose.edge-integration.yml`:
   - `AI_ENGINE_API_KEY=dev_api_test_key_2026`
   - `AI_ENGINE_BRIDGE_API_KEY=dev_bridge_test_key_2026`
3. Restart `bff-backoffice` and `api-gateway` in the edge stack.
4. Repeat the minimum operational smoke section.

## Validation

Recovery is complete when:

- `ai-engine-api` and `ai-engine-stats` return `200` on `/health`
- `ai-engine-stats` returns `200` on `/stats` and `/stats/history` with a valid key
- the edge returns `200` on the four aggregated AI metrics and logs routes

## Rollback or escalation

- if the services stay unhealthy after restart, inspect local compose logs and verify `.env` plus `.env.secrets` resolution
- if auth still fails after restoring the standard dev keys, re-check the effective values loaded by `bff-backoffice`, `api-gateway`, and `ai-engine`
- if the edge passes but direct AI endpoints fail, continue with repository-local AI diagnostics in the `ai-engine` repo

## Related documents

- `docs/operations/runtime-routing-and-service-targeting.md`
- `docs/operations/secrets-distribution-guide.md`
- `docs/guides/capabilities/ai/ai-observability-and-diagnostics.md`
