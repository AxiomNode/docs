# Service Communication Model

Ultima actualizacion: 2026-03-29.

## Flujo sincrono

1. Cliente mobile o backoffice llama a `api-gateway` (:7005).
2. `api-gateway` valida politicas de borde (EDGE_API_TOKEN, CORS, correlation-id).
3. `api-gateway` enruta al BFF correspondiente usando `forwardHttp` de shared-sdk-client.
4. BFF agrega y transforma respuestas consultando microservicios internos.

## Endpoints publicos (api-gateway → BFFs)

### Mobile (via bff-mobile)

| Metodo | Ruta | Destino |
|--------|------|---------|
| GET | `/v1/mobile/games/quiz/random` | quizz:/games/models/random |
| GET | `/v1/mobile/games/wordpass/random` | wordpass:/games/models/random |
| POST | `/v1/mobile/games/quiz/generate` | quizz:/games/generate (120s timeout) |
| POST | `/v1/mobile/games/wordpass/generate` | wordpass:/games/generate (120s timeout) |

### Backoffice (via bff-backoffice)

| Metodo | Ruta | Destino |
|--------|------|---------|
| POST | `/v1/backoffice/auth/session` | users:/users/firebase/session |
| GET | `/v1/backoffice/auth/me` | users:/users/me/profile |
| GET | `/v1/backoffice/users/leaderboard` | users:/users/leaderboard |
| GET | `/v1/backoffice/monitor/stats` | users:/monitor/stats |
| GET | `/v1/backoffice/admin/users/roles` | users:/users/admin/roles |
| PATCH | `/v1/backoffice/admin/users/roles/:uid` | users:/users/admin/roles/:uid |
| GET | `/v1/backoffice/services` | catalogo local de servicios |
| GET | `/v1/backoffice/services/:s/metrics` | servicio:/monitor/stats |
| GET | `/v1/backoffice/services/:s/logs` | servicio:/monitor/logs |
| GET | `/v1/backoffice/services/:s/data` | servicio (segun dataset) |
| GET | `/v1/backoffice/services/:s/catalogs` | servicio:/catalogs |
| POST | `/v1/backoffice/services/:s/generation/process` | servicio:/games/generate/process |
| POST | `/v1/backoffice/services/:s/generation/wait` | servicio:/games/generate/process/wait |
| GET | `/v1/backoffice/services/:s/generation/processes` | servicio:/games/generate/processes |

## Flujo asincrono

- Procesos de generacion batch inician con POST y devuelven taskId.
- Polling con GET `/generation/process/:taskId` para seguimiento.
- Los contratos de eventos se versionan en `contracts-and-schemas/schemas/events/`.

## Reglas de comunicacion

- No exponer microservicios de dominio directamente a internet.
- Evitar llamadas BFF a BFF.
- Definir timeouts y retries por dependencia (`UPSTREAM_TIMEOUT_MS`, `UPSTREAM_GENERATION_TIMEOUT_MS`).
- Incluir `x-correlation-id` en toda peticion (propagado por `extractForwardHeaders` del SDK).
- Headers de autenticacion propagados: `authorization`, `x-firebase-id-token`, `x-dev-firebase-uid`, `x-api-key`.

## Autenticacion entre capas

| Capa | Mecanismo |
|------|-----------|
| Cliente → api-gateway | `Authorization: Bearer EDGE_API_TOKEN` |
| bff-backoffice → ai-engine | `X-API-Key: AI_ENGINE_BRIDGE_API_KEY` |
| microservice-quizz/wordpass → ai-engine | `X-API-Key: AI_ENGINE_API_KEY` |
| Backoffice UI → Firebase | Firebase ID Token |
| bff-backoffice → microservice-users | Forward `x-firebase-id-token` header |
