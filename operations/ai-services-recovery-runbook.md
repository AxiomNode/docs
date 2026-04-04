# AI Services Recovery Runbook (Dev)

Goal: quickly recover `ai-engine-api` and `ai-engine-stats` when they appear `DOWN` or fail with auth errors.

## 1) Startup and health

1. Go to `ai-engine/src`.
2. Ejecutar `docker compose up -d ai-cache ai-stats ai-api`.
3. Verify:
   - `GET http://localhost:7001/health` => `200`
   - `GET http://localhost:7000/health` => `200`

## 2) Minimum operational smoke

1. Obtain effective runtime keys for `ai-stats`:
   - `AI_ENGINE_API_KEY`
   - `AI_ENGINE_BRIDGE_API_KEY`
2. Validate monitoring endpoints:
   - `GET http://localhost:7000/stats` con `X-API-Key` => `200`
   - `GET http://localhost:7000/stats/history?last_n=5` con `X-API-Key` => `200`
3. Validate aggregated consumption through edge:
   - `GET /v1/backoffice/services/ai-engine-api/metrics` => `200`
   - `GET /v1/backoffice/services/ai-engine-api/logs?limit=5` => `200`
   - `GET /v1/backoffice/services/ai-engine-stats/metrics` => `200`
   - `GET /v1/backoffice/services/ai-engine-stats/logs?limit=5` => `200`

## 3) Key rollback (if 401/403 appears)

1. Check for key mismatch between `bff-backoffice` and `ai-engine` runtime.
2. Restore standard dev keys in `platform-infra/environments/dev/docker-compose.edge-integration.yml`:
   - `AI_ENGINE_API_KEY=dev_api_test_key_2026`
   - `AI_ENGINE_BRIDGE_API_KEY=dev_bridge_test_key_2026`
3. Restart `bff-backoffice` and `api-gateway` in edge stack.
4. Repeat section 2 smoke checks.

## 4) Recovery criteria

Recovery is complete when:

- `ai-engine-api` y `ai-engine-stats` responden `200` en `/health`.
- `ai-engine-stats` responde `200` en `/stats` y `/stats/history` con key valida.
- Edge responde `200` en las 4 rutas agregadas de metrics/logs AI.
