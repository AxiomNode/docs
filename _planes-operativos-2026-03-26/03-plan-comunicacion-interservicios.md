# 03. Plan De Comunicacion Entre Servicios

## Objetivo

Estandarizar la comunicacion entre servicios para que sea:

- Segura.
- Observable de extremo a extremo.
- Resiliente ante fallos parciales.
- Facil de evolucionar sin romper consumidores.

---

## Mapa de comunicacion objetivo

1. `backoffice` -> `api-gateway` (edge).
2. `api-gateway` -> `bff-backoffice` o `bff-mobile`.
3. `bff-backoffice` -> `microservice-*`, `ai-engine-*`, `api-gateway` (segun endpoint).
4. `bff-mobile` -> `microservice-quizz` y `microservice-wordpass`.
5. `microservice-quizz/wordpass` -> `ai-engine-api` y/o `ai-engine-stats`.
6. `ai-engine-stats` <-> `ai-engine-api` (cache/monitoring interno).

---

## Contratos y versionado

### Reglas

- Cada endpoint/evento debe tener contrato en `contracts-and-schemas`.
- Breaking changes solo con version nueva.
- Consumer-driven contract tests obligatorios.

### Mejoras

- P0: Publicar matriz productor-consumidor por contrato.
- P0: CI bloquea merges con breaking changes no versionadas.
- P1: Generacion automatica de SDK despues de aprobar contrato.

### KPI

- Incidentes por incompatibilidad de contrato = 0.

---

## Seguridad de comunicacion

### Reglas

- Propagacion consistente de headers:
  - `authorization`
  - `x-firebase-id-token`
  - `x-api-key` (segun servicio)
  - `x-correlation-id`
- Zero trust interno: no asumir red confiable.

### Mejoras

- P0: Middleware comun de forwarding y sanitizacion de headers.
- P0: Tabla de politicas de auth por ruta (quien llama, con que credencial).
- P1: Rotacion de claves y validacion de expiracion en runtime.
- P2: mTLS interno para enlaces criticos.

### KPI

- Errores por header/credencial faltante <= 0.1%.

---

## Resiliencia y calidad de enlace

### Reglas

- Timeout por dependencia y endpoint.
- Retry solo para errores transitorios.
- Circuit breaker y bulkhead en clientes internos.

### Mejoras

- P0: Politica unificada de timeout/retry por tipo de operacion.
- P0: Circuit breaker en llamadas a AI y servicios de datos criticos.
- P1: Dead letter / cola de compensacion para procesos asincronos.
- P2: Chaos testing controlado en staging.

### KPI

- Cascadas de error reducidas >= 80%.
- MTTR de degradaciones interservicio <= 20 min.

---

## Observabilidad extremo a extremo

### Reglas

- Todo request debe llevar `x-correlation-id`.
- Logs estructurados compatibles entre repos.
- Trazas distribuidas obligatorias en hop interservicio.

### Mejoras

- P0: Estandar de log event schema compartido.
- P0: Instrumentacion OpenTelemetry en gateway, BFF y microservicios.
- P1: Dashboard por journey:
  - `backoffice -> bff-backoffice -> microservice -> ai-engine`.
- P1: Alertas por degradacion funcional (no solo 5xx).
- P2: Score de salud por cadena de dependencias.

### KPI

- Cobertura de trazas end-to-end >= 95%.
- Tiempo de causa raiz <= 10 min.

---

## Calidad de datos en transito

### Reglas

- Validacion de payload de entrada/salida en todos los bordes.
- Idempotencia en operaciones de escritura sensibles.

### Mejoras

- P0: Validacion schema estricta en BFF y microservicios.
- P0: Clave de idempotencia para escrituras manuales y eventos.
- P1: Catalogo de errores de datos con codigos estables.
- P2: Monitoreo de drift semantico en payloads AI.

### KPI

- Errores de validacion en runtime < 1%.
- Escrituras duplicadas < 0.1%.

---

## Plan de implantacion (90 dias)

## Fase 1 (0-30 dias)

- Implementar estandar de headers, correlation-id y contratos bloqueantes.
- Activar dashboards minimos por servicio + alertas P0.
- Corregir rutas criticas con fallos de auth interservicio.

## Fase 2 (31-60 dias)

- Incorporar trazas distribuidas y circuit breakers.
- Introducir cache/coalescing en BFF donde corresponda.
- Añadir pruebas de contrato productor-consumidor en CI.

## Fase 3 (61-90 dias)

- Optimizar costo/latencia por journey.
- Ejecutar chaos drills en staging.
- Cerrar backlog P2 y formalizar scorecards operativos por equipo.

---

## Scorecard unico recomendado

Medir semanalmente por servicio y por flujo:

1. Disponibilidad (SLO).
2. P95/P99 latencia.
3. Error rate total y por causa.
4. Calidad de datos (validacion, duplicados, completitud).
5. Eficacia funcional (`requested -> created`, exito por flujo).
6. Eficiencia (CPU, memoria, costo por resultado).
7. Utilidad (acciones operativas resueltas, impacto en usuario/negocio).

Criterio de sistema saludable:

- Todos los servicios en verde en disponibilidad y error rate.
- Al menos 90% de flujos criticos con eficacia dentro de objetivo.
- MTTR dentro de SLA y sin incidentes recurrentes por misma causa raiz.

---

## Checklist de implantacion (comunicacion)

### Contratos

- [ ] Matriz productor-consumidor publicada.
- [ ] Breaking changes bloqueadas sin version.
- [ ] Contract tests en CI de productores y consumidores.

### Seguridad de enlace

- [ ] Politica de headers obligatorios implementada en edge/BFF.
- [ ] Tabla de auth por ruta documentada y vigente.
- [ ] Rotacion de claves automatizada en entorno objetivo.

### Resiliencia

- [ ] Timeouts por dependencia configurados.
- [ ] Retry policy separada por error transitorio/permanente.
- [ ] Circuit breaker activo en llamadas criticas.

### Observabilidad

- [ ] Correlation id propagado en 100% de hops.
- [ ] Trazas distribuidas activas en journeys criticos.
- [ ] Alertas funcionales y tecnicas con runbook.

### Calidad de datos en transito

- [ ] Validacion schema en bordes de entrada/salida.
- [ ] Idempotencia en operaciones sensibles.
- [ ] Catalogo de errores estable y versionado.
