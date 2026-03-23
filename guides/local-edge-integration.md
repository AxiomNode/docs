# Local Edge Integration

## Objetivo

Levantar `api-gateway`, `bff-mobile` y `bff-backoffice` en una sola composicion para pruebas integradas locales.

## Prerrequisitos

1. Tener activos los microservicios de dominio en host:
   - `microservice-quizz` en `7100`
   - `microservice-wordpass` en `7101`
   - `microservice-users` en `7102`
2. Docker Desktop corriendo.
3. Secretos de entorno dev en repo privado `secrets`.
4. Ejecutar `node scripts/export-secrets-map.mjs dev` desde `secrets`.

## Levantar edge layer

Desde `platform-infra/environments/dev`:

```bash
docker compose -f docker-compose.edge-integration.yml up -d --build
```

## Comprobaciones rapidas

```bash
curl http://localhost:7005/health
curl "http://localhost:7005/v1/mobile/games/quiz/random?language=es"
curl "http://localhost:7005/v1/backoffice/users/leaderboard?limit=5"
```

Si `EDGE_API_TOKEN` esta configurado en gateway, incluir:

```bash
-H "Authorization: Bearer <EDGE_API_TOKEN>"

`EDGE_API_TOKEN` se carga desde `api-gateway/src/.env.secrets`, copiado desde `secrets/runtime/dev/api-gateway.env`.

## Smoke test automatizado

```bash
bash scripts/smoke-edge.sh
```
```

## Apagado

```bash
docker compose -f docker-compose.edge-integration.yml down
```
