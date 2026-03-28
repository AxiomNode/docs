# 04. Checklist Ejecutable Del Primer Plan (Servicios)

Objetivo de esta hoja:

- Arrancar ya el Plan 1 (servicios) con una lista operativa que se pueda marcar.
- Enfocar primero en P0 de seguridad, observabilidad y eficacia de flujos criticos.

## Bloque 1: baseline y ownership

- [x] Asignar owner por servicio (nombre y backup).
- [x] Capturar baseline inicial por servicio:
  - [x] disponibilidad 24h,
  - [x] p95 latencia,
  - [x] error rate,
  - [x] conversion funcional (si aplica).
- [x] Publicar dashboard temporal de baseline.

## Bloque 2: autenticacion y forwarding

- [x] Verificar forwarding de `authorization` en todos los hops.
- [x] Verificar forwarding de `x-firebase-id-token` donde aplique.
- [x] Verificar forwarding de `x-api-key` en rutas AI.
- [x] Añadir test de regresion por cada cabecera critica.

## Bloque 3: resiliencia minima

- [x] Configurar timeout por dependencia en gateway y BFF.
- [x] Activar circuit breaker para AI y servicios de datos (quiz/wordpass en rutas de generacion).
- [x] Bloquear reintentos en errores 4xx no recuperables (401/403 en AI para evitar loops).

## Bloque 4: observabilidad estandar

- [x] Log schema unificado en servicios runtime.
- [x] Metricas Prometheus por endpoint clave.
- [x] Correlation id en logs y respuestas.
- [x] Alertas P0 con runbook por servicio.

## Bloque 5: flujo critico quiz/wordpass

- [x] Smoke de credenciales AI al arranque.
- [x] Abort batch si smoke falla.
- [x] Taxonomia de errores operativa implementada (503 cuando el circuito auth esta abierto).
- [x] Dashboard de conversion `requested -> created`.

## Bloque 6: cierre de iteracion

- [x] Ejecutar build/test/contract tests de servicios tocados (quiz y wordpass en verde).
- [x] Comparar KPI baseline vs KPI post-cambio.
- [x] Listar deuda P1/P2 priorizada por impacto.
- [x] Publicar informe de resultados de iteracion.

## Criterios de salida de este primer plan

- [x] Cero incidentes por headers faltantes en rutas criticas.
- [x] Todas las rutas operativas clave con observabilidad minima.
- [x] Flujos de generacion AI con resultado medible y trazable.
- [x] Flujos criticos tienen trazabilidad end-to-end.
- [x] Estado de cada servicio clasificable como verde/amarillo/rojo con evidencia.

## Tabla de control sugerida

| Item | Responsable | Estado | Evidencia |
|---|---|---|---|
| Baseline KPI por servicio | Squad SRE Platform | [x] | Informe iteracion 01 con snapshot runtime y comparativa pre/post |
| Forwarding de headers criticos | Squad Edge Platform | [x] | Forwarding unificado en `@axiomnode/shared-sdk-client/proxy` (`authorization`, `x-firebase-id-token`, `x-api-key`, `x-correlation-id`) + propagacion de `x-correlation-id` en fetch directo de bff-backoffice + regresion en `api-gateway/src/tests/proxy.test.ts`, `bff-mobile/src/tests/mobile.test.ts` y `bff-backoffice/src/tests/backoffice.test.ts` (verde) + smoke runtime `platform-infra/environments/dev/scripts/smoke-edge.sh` OK |
| Circuit breaker y timeout policy | Squad Edge Platform | [x] | Breaker auth AI activo + timeout policy en api-gateway, bff-backoffice y bff-mobile |
| Observabilidad minima activa | Squad AI Observability | [x] | Logs estructurados con `correlation_id` + metricas Prometheus (`requests_total`, `errors_total`, `latency_ms_bucket`, `inflight_requests`) en edge/BFF + endpoint `/metrics` activo en quiz/wordpass + propagacion `traceparent/tracestate/baggage` + reglas P0 en `observability-platform/alerts/p0-services-alerts.rules.yml` + validacion runtime 2026-03-28 (200 en `/metrics`, `/monitor/stats`, `/monitor/logs` para `api-gateway`, `bff-mobile`, `bff-backoffice`, `microservice-users`, `microservice-quiz`, `microservice-wordpass`, y 200 en rutas agregadas `/v1/backoffice/services/:service/metrics|logs`) |
| Trazabilidad end-to-end | Squad Edge Platform | [x] | Validacion runtime 2026-03-28 con `x-correlation-id` comun observado en logs de 3 hops por flujo: `api-gateway -> bff-backoffice -> microservice-users` (leaderboard), `api-gateway -> bff-mobile -> microservice-quiz` (quiz random), `api-gateway -> bff-mobile -> microservice-wordpass` (wordpass random) |

## Matriz de estado por servicio (snapshot 2026-03-28)

| Servicio | Estado | Evidencia |
|---|---|---|
| api-gateway | verde | `200` en `/health`, `/metrics`, `/monitor/stats`, `/monitor/logs`; rutas edge criticas `200` |
| bff-mobile | verde | `200` en `/health`, `/metrics`, `/monitor/stats`, `/monitor/logs`; flujo random quiz/wordpass `200` |
| bff-backoffice | verde | `200` en `/health`, `/metrics`, `/monitor/stats`, `/monitor/logs`; agregacion servicios `200` |
| microservice-users | verde | `200` en `/health`, `/metrics`, `/monitor/stats`, `/monitor/logs`; leaderboard via edge `200` |
| microservice-quiz | verde | `200` en `/health`, `/metrics`, `/monitor/stats`, `/monitor/logs`; metrics via edge `200` |
| microservice-wordpass | verde | `200` en `/health`, `/metrics`, `/monitor/stats`, `/monitor/logs`; metrics via edge `200` |
| ai-engine-stats | rojo | endpoint en `7000` no disponible en snapshot (`DOWN`) |
| ai-engine-api | rojo | endpoint en `7001` no disponible en snapshot (`DOWN`) |
| llama-server | amarillo | servicio accesible en `11434` pero sin contrato de observabilidad estandar (`/health|/metrics|/monitor/*` devuelven `404`) |

### Nota de ejecucion runtime

- 2026-03-28: primer intento bloqueado por entorno offline (`WinError 10061` en puertos 7005, 7010, 7011, 7100, 7101).
- 2026-03-28: validacion completada con `platform-infra/environments/dev/scripts/smoke-edge.sh` y resultado OK para forwarding de `authorization`, `x-correlation-id`, `x-firebase-id-token`, `x-api-key`.

## Mini-iteracion AI (rojo -> verde) checklist ejecutable

Objetivo:

- Llevar `ai-engine-api` y `ai-engine-stats` de rojo a verde con evidencia runtime + consumo agregado via edge.

### Bloque A: arranque y disponibilidad AI

- [ ] Levantar `ai-engine-api` en `7001` con endpoint `/health` operativo.
- [ ] Levantar `ai-engine-stats` en `7000` con endpoint `/health` operativo.
- [ ] Validar `200` en `/monitor/stats` y `/monitor/logs?limit=5` para ambos servicios.

### Bloque B: autenticacion y consumo agregado

- [ ] Alinear `AI_ENGINE_API_KEY` y `AI_ENGINE_BRIDGE_API_KEY` en BFF y microservicios consumidores.
- [ ] Validar `200` en edge para `/v1/backoffice/services/ai-engine-api/metrics`.
- [ ] Validar `200` en edge para `/v1/backoffice/services/ai-engine-api/logs?limit=5`.
- [ ] Validar `200` en edge para `/v1/backoffice/services/ai-engine-stats/metrics`.
- [ ] Validar `200` en edge para `/v1/backoffice/services/ai-engine-stats/logs?limit=5`.

### Bloque C: runbook y alertado

- [ ] Publicar runbook corto de recuperacion AI (arranque, smoke, rollback de keys).
- [ ] Enlazar runbook en alerta P0 de servicios AI.
- [ ] Ejecutar simulacion de incidente y registrar MTTR observado.

### Criterio de salida mini-iteracion AI

- [ ] `ai-engine-api` clasificado en verde con evidencia runtime.
- [ ] `ai-engine-stats` clasificado en verde con evidencia runtime.
- [ ] Consumo agregado via edge validado con estado 200 para metrics/logs AI.

### Tabla de control mini-iteracion AI

| Item | Responsable | Estado | Evidencia |
|---|---|---|---|
| Arranque ai-engine-api y ai-engine-stats | Squad AI Platform | [ ] |  |
| Alineacion de keys AI | Squad Edge Platform | [ ] |  |
| Smoke agregado por edge (metrics/logs AI) | Squad Backoffice Backend | [ ] |  |
| Runbook + alerta P0 AI | Squad AI Observability | [ ] |  |
