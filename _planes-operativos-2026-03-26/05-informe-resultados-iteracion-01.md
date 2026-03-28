# 05. Informe De Resultados De Iteracion 01 (Servicios)

Fecha de corte: 2026-03-26

## 1) Alcance evaluado

Se evaluo la iteracion ejecutada sobre:

- `api-gateway`
- `bff-backoffice`
- `bff-mobile`
- `microservice-quiz`
- `microservice-wordpass`

Con foco en P0:

- forwarding/auth en hops criticos,
- timeout policy en edge/BFF,
- circuit breaker y smoke para flujos AI quiz/wordpass,
- dashboard de conversion `requested -> created` en backoffice.

## 2) Evidencia tecnica

### 2.1 Validacion de build/tests

- `api-gateway`: build OK, tests OK.
- `bff-backoffice`: build OK, tests OK.
- `bff-mobile`: build OK, tests OK.
- `backoffice`: build OK, tests OK.
- `microservice-quiz`: build OK, tests OK.
- `microservice-wordpass`: build OK, tests OK.

### 2.2 Snapshot runtime (health y monitor)

Health endpoints verificados en `localhost`:

- `7005` (api-gateway): OK
- `7011` (bff-backoffice): OK
- `7010` (bff-mobile): OK
- `7100` (microservice-quiz): OK
- `7101` (microservice-wordpass): OK

KPI observados en monitor (post-redeploy de quiz/wordpass + corrida final `count=3` por servicio):

- `microservice-quiz`
  - proceso final `requested=3`, `processed=3`, `created=1`, `failed=2`
  - `processes.requestedTotal=21`
  - `processes.createdTotal=1`
  - `generation.attemptsTotal=21`
  - `generation.successRatio=0.0476`
  - `outboundByOperation`: `generate=200` (1), `generate=422` (2), `generate=500` (6)
- `microservice-wordpass`
  - proceso final `requested=3`, `processed=3`, `created=1`, `failed=0`, `duplicates=2`
  - `processes.requestedTotal=21`
  - `processes.createdTotal=1`
  - `generation.attemptsTotal=21`
  - `generation.successRatio=0.0476`
  - `outboundByOperation`: `generate=200` (1), `generate=500` (6)

Hallazgo clave de calidad de payload (quiz):

- Se registraron errores `422` con detalle `Unknown game_type 'educational-game'` en respuestas del AI Engine durante la corrida final.
- Impacto: reduce conversion efectiva de quiz aun con auth ya establecida.
- Mitigacion aplicada en codigo: normalizacion defensiva de aliases de `game_type` en `ai-engine` (`educational-game -> quiz`).
- Estado tras mitigacion: persisten `422` por validacion de campos faltantes en payload del modelo (no por `game_type`), por lo que la deuda pasa a calidad estructural de salida LLM.

## 3) Comparativa KPI baseline vs post-cambio

## 3.1 Resiliencia edge/BFF

- Baseline (pre): politica de timeout no homogenea en hops edge/BFF.
- Post: timeout estandar aplicado con `UPSTREAM_TIMEOUT_MS` y mapping de expiracion a `504`.
- Resultado: mejora estructural aplicada y validada por build/tests.

## 3.2 Flujo AI quiz/wordpass

- Baseline (pre): incidentes por auth faltante en integraciones AI; fallos de generacion no acotados.
- Post (codigo): breaker + smoke + fail-fast 503 implementados.
- Post (runtime observado): desaparecen `403` en operaciones AI para quiz/wordpass tras alinear key efectiva en compose de ambos microservicios.
- Resultado: resiliencia de control implementada y autenticacion operativa recuperada; queda pendiente consolidar eficacia con nueva corrida batch.

## 3.3 Visibilidad operativa

- Baseline (pre): faltaba dashboard de conversion en overview.
- Post: dashboard `requested -> created` desplegado en backoffice.
- Resultado: mejora completada (observabilidad funcional desde UI).

## 4) Conclusiones de iteracion

- La capa edge/BFF queda endurecida (timeouts y error semantics consistentes).
- El control de dano en quiz/wordpass esta implementado y la causa raiz de `403` quedo corregida en runtime.
- La eficacia de generacion ya es medible y trazable (se obtuvo `created=1` en cada servicio en la corrida final corta), pero aun esta lejos del objetivo operativo por baja conversion global (`1/21` por servicio, ~`4.76%`).
- En quiz persiste deuda funcional en el payload del modelo (respuestas con `game_type=educational-game`) que provoca rechazos `422`.

## 5) Deuda priorizada (P1/P2)

Prioridad inmediata (operativa):

1. Endurecer post-procesado de salida LLM para quiz (campos obligatorios y estructura de `questions`) para reducir `422` remanentes.
2. Ejecutar corrida batch completa post-fix de payload y registrar nuevamente `requested -> created` por servicio.
3. Validar que `successRatio` y `failureRatio` se estabilizan en ventana operativa (sin regresion de auth).

P1 tecnico:

1. Exponer explicitamente `aiAuthCircuit` en snapshot de monitor y panel para diagnostico directo de apertura/cierre del circuito.
2. Añadir alerta de degradacion por `createdTotal/requestedTotal` por debajo de umbral.
3. Incorporar comparativa temporal (ventana ultima hora/dia) en panel de overview.

P2 tecnico:

1. Añadir reporte automatico de costo por item generado y ratio de desperdicio por intentos fallidos.
2. Integrar trazas distribuidas en todo el camino `gateway -> bff -> microservice -> ai-engine` para MTTD menor.

## 6) Decision de salida

Estado de iteracion 01: **exitosa con seguimiento**.

- Exitosa en hardening estructural, observabilidad de UI y recuperacion de autenticacion AI.
- Con seguimiento porque la conversion global sigue baja y quiz mantiene rechazos `422` por payload incompleto del modelo.

Recomendacion: abrir iteracion corta enfocada en calidad estructural de payload quiz (titulo, preguntas y campos obligatorios) y cerrar con nueva medicion de conversion sostenida post-fix.

## 7) Avance Bloque A (fundacion operativa)

- Owner y backup tecnico definidos para todos los servicios del Plan 1.
- Responsabilidad principal publicada en README de cada servicio del alcance.
- SLO y error budget mensual acordados por servicio (edge 99.9%, internos 99.5%).

## 8) Avance Bloque B (forwarding de headers criticos)

- Forwarding de headers criticos unificado en edge/BFF: `authorization`, `x-firebase-id-token`, `x-api-key`, `x-correlation-id`.
- Ajuste en `bff-backoffice` para incluir `x-correlation-id` en fetch directo a servicios aguas abajo.
- Regresion automatizada agregada y en verde en:
  - `api-gateway/src/tests/proxy.test.ts`
  - `bff-mobile/src/tests/mobile.test.ts`
  - `bff-backoffice/src/tests/backoffice.test.ts`

## 9) Estado de validacion runtime del criterio de headers

- 2026-03-28: se intento ejecutar smoke e2e en localhost para validar ausencia de incidentes por headers faltantes.
- Resultado del intento: bloqueado por entorno no disponible (`WinError 10061`, connection refused) en `7005`, `7010`, `7011`, `7100`, `7101`.
- Accion preparada: `platform-infra/environments/dev/scripts/smoke-edge.sh` actualizado para incluir forwarding de cabeceras criticas (`authorization`, `x-correlation-id`, `x-firebase-id-token`, `x-api-key`) cuando el entorno este levantado.

## 10) Cierre de validacion runtime del criterio de headers

- 2026-03-28: con entorno disponible, se ejecuto `platform-infra/environments/dev/scripts/smoke-edge.sh` con resultado **OK**.
- El smoke verifico flujo edge/BFF en rutas GET y POST con envio de headers criticos y sin fallos recurrentes atribuibles a headers faltantes.

## 11) Cierre de validacion runtime de metricas y logs

- 2026-03-28: validacion runtime directa completada con estado 200 en `/metrics`, `/monitor/stats` y `/monitor/logs` para:
  - `api-gateway` (`7005`)
  - `bff-mobile` (`7010`)
  - `bff-backoffice` (`7011`)
  - `microservice-users` (`7102`)
  - `microservice-quiz` (`7100`)
  - `microservice-wordpass` (`7101`)
- 2026-03-28: validacion agregada via edge completada con estado 200 en:
  - `/v1/backoffice/services/:service/metrics`
  - `/v1/backoffice/services/:service/logs?limit=5`
  para `api-gateway`, `bff-mobile`, `microservice-users`, `microservice-quiz`, `microservice-wordpass`.

## 12) Cierre de trazabilidad end-to-end en flujos criticos

- 2026-03-28: validacion runtime con `x-correlation-id` y contexto de traza en requests edge.
- Evidencia en logs estructurados por hop con el mismo `correlation_id` para:
  - `api-gateway -> bff-backoffice -> microservice-users` sobre `/v1/backoffice/users/leaderboard`.
  - `api-gateway -> bff-mobile -> microservice-quiz` sobre `/v1/mobile/games/quiz/random`.
  - `api-gateway -> bff-mobile -> microservice-wordpass` sobre `/v1/mobile/games/wordpass/random`.
- Resultado: criterio de trazabilidad end-to-end cerrado para los flujos criticos del alcance.

## 13) Clasificacion verde/amarillo/rojo por servicio (cierre de criterio)

Snapshot runtime realizado el 2026-03-28 con verificacion directa de `/health`, `/metrics`, `/monitor/stats`, `/monitor/logs` y rutas criticas via edge.

- Verde:
  - `api-gateway`
  - `bff-mobile`
  - `bff-backoffice`
  - `microservice-users`
  - `microservice-quiz`
  - `microservice-wordpass`
- Amarillo:
  - `llama-server` (servicio accesible, pero sin endpoints de observabilidad estandar en este entorno)
- Rojo:
  - `ai-engine-stats` (down en puerto `7000`)
  - `ai-engine-api` (down en puerto `7001`)

Resultado: criterio de salida "estado de cada servicio clasificable como verde/amarillo/rojo con evidencia" **cerrado**.

## 14) Recomendacion operativa corta (rojo -> verde en AI)

Objetivo de la siguiente iteracion corta: llevar `ai-engine-api` y `ai-engine-stats` de **rojo** a **verde** en entorno local estandar.

Acciones minimas propuestas:

1. Levantar `ai-engine-api` y `ai-engine-stats` como parte de un compose dev estable (puertos `7001` y `7000`) con healthchecks obligatorios.
2. Alinear llaves de integracion (`AI_ENGINE_API_KEY` y `AI_ENGINE_BRIDGE_API_KEY`) entre `bff-backoffice`, `microservice-quiz` y `microservice-wordpass` para evitar 401/403 intermitentes.
3. Exponer y validar en runtime `200` para `/health`, `/monitor/stats` y `/monitor/logs` en ambos servicios AI.
4. Ejecutar smoke de consumo agregado por edge (`/v1/backoffice/services/ai-engine-api|ai-engine-stats/metrics|logs`) y registrar evidencia en checklist.
5. Definir runbook de recuperacion rapida para AI (arranque, smoke, rollback de keys) y asociarlo a alerta P0.

Condicion de cierre sugerida para esta mini-iteracion:

- `ai-engine-api` y `ai-engine-stats` clasificados en verde con evidencia runtime + smoke agregado en edge.

Checklist ejecutable asociada:

- Ver seccion `Mini-iteracion AI (rojo -> verde) checklist ejecutable` en `04-checklist-primer-plan-servicios.md`.

## 15) Avance mini-iteracion AI (Bloque A completado)

Fecha de ejecucion: 2026-03-28.

Estado actual del Bloque A (arranque y disponibilidad): **completado**.

Evidencia runtime validada:

- `ai-engine-api` (`7001`): `GET /health` -> `200`.
- `ai-engine-stats` (`7000`): `GET /health` -> `200`.
- `ai-engine-stats` (`7000`): `GET /stats` -> `200` con `X-API-Key` valida.
- `ai-engine-stats` (`7000`): `GET /stats/history?last_n=5` -> `200` con `X-API-Key` valida.

Hallazgo operativo para Bloque B:

- `ai-engine-stats` acepta para endpoints de monitoreo llaves de `AI_ENGINE_API_KEY` o `AI_ENGINE_BRIDGE_API_KEY`.
- En pruebas iniciales se observo `403` al usar valores no alineados con el runtime efectivo del contenedor.
- La alineacion explicita de keys en consumidores edge/BFF se valido y quedo cerrada en Bloque B.

## 16) Avance mini-iteracion AI (Bloque B completado)

Fecha de ejecucion: 2026-03-28.

Estado actual del Bloque B (autenticacion y consumo agregado): **completado**.

Evidencia runtime validada en edge (`api-gateway:7005`):

- `GET /v1/backoffice/services/ai-engine-api/metrics` -> `200`.
- `GET /v1/backoffice/services/ai-engine-api/logs?limit=5` -> `200`.
- `GET /v1/backoffice/services/ai-engine-stats/metrics` -> `200`.
- `GET /v1/backoffice/services/ai-engine-stats/logs?limit=5` -> `200`.

Resultado operativo:

- Queda confirmada la alineacion efectiva de `AI_ENGINE_API_KEY` y `AI_ENGINE_BRIDGE_API_KEY` en el consumo agregado de backoffice hacia servicios AI para este entorno dev.

## 17) Avance mini-iteracion AI (Bloque C completado)

Fecha de ejecucion: 2026-03-28.

Estado actual del Bloque C (runbook y alertado): **completado**.

Evidencia de cierre:

- Runbook corto publicado: `docs/operations/ai-services-recovery-runbook.md`.
- Alerta P0 AI enlazada con runbook: `AxiomNodeAIServicesDown` en `observability-platform/alerts/p0-services-alerts.rules.yml`.
- Simulacion controlada de incidente (restart de `ai-stats` y `ai-api`) con recuperacion a `200` en ambos health checks.
- MTTR observado en simulacion: `10s`.

## 18) Cierre mini-iteracion AI (rojo -> verde)

Resultado: **mini-iteracion AI cerrada**.

- `ai-engine-api`: verde (evidencia runtime y consumo agregado via edge en `200`).
- `ai-engine-stats`: verde (evidencia runtime y consumo agregado via edge en `200`).
- Criterio de salida mini-AI: cumplido en checklist operativo.
