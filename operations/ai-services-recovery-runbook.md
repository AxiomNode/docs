# AI Services Recovery Runbook (Dev)

Objetivo: recuperar rapido `ai-engine-api` y `ai-engine-stats` cuando aparecen `DOWN` o con errores de autenticacion.

## 1) Arranque y health

1. Posicionarse en `ai-engine/src`.
2. Ejecutar `docker compose up -d ai-cache ai-stats ai-api`.
3. Verificar:
   - `GET http://localhost:7001/health` => `200`
   - `GET http://localhost:7000/health` => `200`

## 2) Smoke operativo minimo

1. Obtener keys efectivas del runtime de `ai-stats`:
   - `AI_ENGINE_API_KEY`
   - `AI_ENGINE_BRIDGE_API_KEY`
2. Validar endpoints de monitoreo:
   - `GET http://localhost:7000/stats` con `X-API-Key` => `200`
   - `GET http://localhost:7000/stats/history?last_n=5` con `X-API-Key` => `200`
3. Validar consumo agregado por edge:
   - `GET /v1/backoffice/services/ai-engine-api/metrics` => `200`
   - `GET /v1/backoffice/services/ai-engine-api/logs?limit=5` => `200`
   - `GET /v1/backoffice/services/ai-engine-stats/metrics` => `200`
   - `GET /v1/backoffice/services/ai-engine-stats/logs?limit=5` => `200`

## 3) Rollback de keys (si hay 401/403)

1. Verificar desalineacion entre `bff-backoffice` y runtime real de `ai-engine`.
2. Restaurar keys dev estandar en `platform-infra/environments/dev/docker-compose.edge-integration.yml`:
   - `AI_ENGINE_API_KEY=dev_api_test_key_2026`
   - `AI_ENGINE_BRIDGE_API_KEY=dev_bridge_test_key_2026`
3. Reiniciar `bff-backoffice` y `api-gateway` del stack edge.
4. Repetir smoke de la seccion 2.

## 4) Criterio de recuperacion

Se considera recuperado cuando:

- `ai-engine-api` y `ai-engine-stats` responden `200` en `/health`.
- `ai-engine-stats` responde `200` en `/stats` y `/stats/history` con key valida.
- Edge responde `200` en las 4 rutas agregadas de metrics/logs AI.
