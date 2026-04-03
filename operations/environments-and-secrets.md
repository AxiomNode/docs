# Environments And Secrets

Ultima actualizacion: 2026-03-29.

## Configuracion por entorno

- Variables no sensibles en `.env` y `.env.example` (versionado).
- Secretos en `.env.secrets` inyectados desde el repo privado `secrets`.
- Nombres y claves consistentes entre servicios dentro de cada distribucion (dev/stg/pro).

## Politica de secretos

- Rotacion periodica via `secrets/scripts/bootstrap-secrets.mjs`.
- No commitear secretos en repositorios de runtime (`.env.secrets` en `.gitignore`).
- Validacion automatica con `validate-secrets-sync.mjs` y `validate-no-placeholders.mjs`.
- Auditoria de hardcoded secrets con `audit-hardcoded-secrets.mjs`.

## Tokens compartidos (mismos dentro de cada distribucion)

| Token | Servicios que lo usan |
|-------|----------------------|
| `API_TOKEN` | microservice-quizz, microservice-wordpass, ai-engine |
| `AI_ENGINE_API_KEY` | microservice-quizz, microservice-wordpass, bff-backoffice, ai-engine |
| `AI_ENGINE_GAMES_API_KEY` | ai-engine |
| `AI_ENGINE_BRIDGE_API_KEY` | bff-backoffice, ai-engine |
| `AI_ENGINE_STATS_API_KEY` | ai-engine |
| `INTERNAL_API_TOKEN` | api-gateway, bff-mobile, bff-backoffice |
| `PRIVATE_DOCS_TOKEN` | microservice-quizz, microservice-wordpass, microservice-users |

## Variables por servicio

### api-gateway (:7005)

| Variable | Tipo | Default |
|----------|------|---------|
| `SERVICE_NAME` | string | api-gateway |
| `SERVICE_PORT` | int | 7005 |
| `ALLOWED_ORIGINS` | string | http://localhost:3000 |
| `BFF_MOBILE_URL` | url | http://localhost:7010 |
| `BFF_BACKOFFICE_URL` | url | http://localhost:7011 |
| `UPSTREAM_TIMEOUT_MS` | int | 15000 |
| `UPSTREAM_GENERATION_TIMEOUT_MS` | int | 120000 |
| `EDGE_API_TOKEN` | secret | (desde .env.secrets) |
| `METRICS_LOG_BUFFER_SIZE` | int | 1000 |

### bff-mobile (:7010)

| Variable | Tipo | Default |
|----------|------|---------|
| `SERVICE_NAME` | string | bff-mobile |
| `SERVICE_PORT` | int | 7010 |
| `QUIZZ_SERVICE_URL` | url | http://localhost:7100 |
| `WORDPASS_SERVICE_URL` | url | http://localhost:7101 |
| `UPSTREAM_TIMEOUT_MS` | int | 15000 |
| `UPSTREAM_GENERATION_TIMEOUT_MS` | int | 120000 |

### bff-backoffice (:7011)

| Variable | Tipo | Default |
|----------|------|---------|
| `SERVICE_NAME` | string | bff-backoffice |
| `SERVICE_PORT` | int | 7011 |
| `USERS_SERVICE_URL` | url | (requerido) |
| `QUIZZ_SERVICE_URL` | url | http://localhost:7100 |
| `WORDPASS_SERVICE_URL` | url | http://localhost:7101 |
| `BFF_MOBILE_URL` | url | http://localhost:7010 |
| `API_GATEWAY_URL` | url | http://localhost:7005 |
| `AI_ENGINE_STATS_URL` | url | http://localhost:7000 |
| `AI_ENGINE_API_URL` | url | http://localhost:7001 |
| `AI_ENGINE_BRIDGE_API_KEY` | secret | (desde .env.secrets) |
| `AI_ENGINE_API_KEY` | secret | (desde .env.secrets) |

### microservice-quizz (:7100) / microservice-wordpass (:7101)

| Variable | Tipo | Default |
|----------|------|---------|
| `SERVICE_PORT` | int | 7100 / 7101 |
| `DATABASE_URL` | string | (requerido) |
| `AI_ENGINE_BASE_URL` | url | http://localhost:7001 |
| `AI_ENGINE_GENERATION_ENDPOINT` | string | /generate/quiz o /generate/word-pass |
| `AI_ENGINE_INGEST_ENDPOINT` | string | /ingest/quiz o /ingest/word-pass |
| `AI_ENGINE_API_KEY` | secret | (desde .env.secrets) |
| `PRIVATE_DOCS_TOKEN` | secret | (desde .env.secrets) |
| `BATCH_GENERATION_*` | varios | Ver config.ts |

### microservice-users (:7102)

| Variable | Tipo | Default |
|----------|------|---------|
| `SERVICE_PORT` | int | 7102 |
| `DATABASE_URL` | string | (requerido) |
| `FIREBASE_CREDENTIALS_JSON` | json | (o FIREBASE_PROJECT_ID + CLIENT_EMAIL + PRIVATE_KEY) |
| `FIREBASE_STRICT_AUTH` | bool | true |
| `SUPERADMIN_FIREBASE_UID` | string | (opcional) |
| `PRIVATE_DOCS_TOKEN` | secret | (desde .env.secrets) |

### backoffice (:7080)

| Variable | Tipo | Default |
|----------|------|---------|
| `VITE_API_BASE_URL` | url | http://localhost:7005 |
| `VITE_EDGE_API_TOKEN` | secret | (desde .env.secrets) |
| `VITE_AUTH_MODE` | string | - |
| `VITE_FIREBASE_*` | varios | Firebase config |
| `VITE_ADMIN_DEV_UID` | string | (solo dev) |

### ai-engine (:7000/:7001)

| Variable | Tipo | Default |
|----------|------|---------|
| `API_TOKEN` | secret | (desde .env.secrets) |
| `AI_ENGINE_API_KEY` | secret | (desde .env.secrets) |
| `AI_ENGINE_GAMES_API_KEY` | secret | (desde .env.secrets) |
| `AI_ENGINE_BRIDGE_API_KEY` | secret | (desde .env.secrets) |
| `AI_ENGINE_STATS_API_KEY` | secret | (desde .env.secrets) |

## Variable huerfana conocida

`SERVICE_COMMUNICATION_TOKEN`: generado y validado por scripts de secrets, pero **no consumido por ningun servicio en runtime**. Candidato a eliminacion en proxima rotacion.
