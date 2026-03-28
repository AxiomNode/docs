# 01. Planes Por Servicio

## Estandar minimo obligatorio para todos los servicios

### Responsabilidad

- Definir una sola responsabilidad principal por servicio (evitar logica cruzada).
- Publicar contrato de entrada/salida versionado.

### Eficacia

- SLO disponibilidad mensual >= 99.5% (servicios internos) y >= 99.9% (edge).
- Error budget explicito por servicio.

### Eficiencia

- P95 latencia por endpoint clave con objetivo fijado.
- Presupuesto maximo de CPU y memoria por request.

### Observabilidad

- Logs estructurados JSON con `correlation_id`, `service`, `route`, `status_code`, `duration_ms`, `error_code`.
- Metricas Prometheus por endpoint: `requests_total`, `errors_total`, `latency_ms_bucket`, `inflight_requests`.
- Trazas OpenTelemetry en llamadas interservicio.
- Alertas con runbook asociado.

---

## backoffice

### Rol principal

- Consola operacional para supervision, accion y mantenimiento seguro del ecosistema.

### Riesgos actuales

- Riesgo de degradacion UX si no hay estados de error consistentes por dependencia.
- Acoplamiento visual/operativo a cambios de schema en BFF.

### Plan de mejoras

- P0: Definir estado operacional unico por card (`healthy`, `degraded`, `down`, `unknown`) con semaforo consistente.
- P0: Establecer politica de mensajes de error orientada a accion (causa probable + siguiente paso).
- P1: Telemetria de acciones de operador (refresh, insert, delete, hotfix) con audit trail.
- P1: Modo "degradado" cuando fallan dependencias, sin bloquear todo el panel.
- P2: Pruebas e2e criticas (overview, mutaciones manuales, auth/roles) en pipeline.

### KPI objetivo

- Tasa de accion completada por operador >= 98%.
- Tiempo medio de diagnostico desde UI <= 2 min.
- Errores UI no manejados = 0.

### Criterio de exito

- Operador puede identificar servicio fallando y ejecutar accion de recuperacion sin abrir logs externos.

---

## api-gateway

### Rol principal

- Punto unico de entrada edge, seguridad base, enrutado y politicas transversales.

### Riesgos actuales

- Riesgo de forwarding heterogeneo (headers/metodos) entre rutas.
- Riesgo de propagacion incompleta de contexto de observabilidad.

### Plan de mejoras

- P0: Unificar middleware de propagacion de headers (`authorization`, `x-firebase-id-token`, `x-api-key`, `x-correlation-id`).
- P0: Politica consistente de timeouts/retries/circuit breaker por upstream.
- P1: Rate limit por ruta y por identidad de cliente.
- P1: Mapeo estandar de errores (4xx/5xx) con `error_code` estable.
- P2: Canary release para rutas criticas con rollback automatico.

### KPI objetivo

- P95 latencia gateway <= 60 ms (sin computo upstream).
- 5xx propios del gateway <= 0.2%.
- Cobertura de trazas con `correlation_id` = 100%.

### Criterio de exito

- Cualquier error interservicio es trazable en menos de 1 minuto desde un unico ID.

---

## bff-backoffice

### Rol principal

- Agregacion operacional para panel administrativo y proxy seguro hacia servicios internos.

### Riesgos actuales

- Riesgo de degradacion por fallos en servicios agregados.
- Riesgo de errores por headers de autenticacion incompletos.

### Plan de mejoras

- P0: Contrato estricto de forwarding de headers por tipo de servicio (incluyendo AI keys cuando aplique).
- P0: Devolver respuestas parciales con estado por dependencia, evitando fallo total del panel.
- P1: Cache corta (5-15s) en metricas agregadas para reducir carga.
- P1: Health score agregado ponderado por criticidad.
- P2: Endpoint de diagnostico con dependencias y ultima causa raiz detectada.

### KPI objetivo

- Error de agregacion por dependencia <= 1%.
- Tiempo de respuesta `/services/:service/metrics` P95 <= 250 ms.
- Incidentes por headers faltantes = 0.

### Criterio de exito

- El backoffice siempre muestra estado util incluso con una o mas dependencias caidas.

---

## bff-mobile

### Rol principal

- Fachada optimizada para clientes moviles (payloads compactos, baja latencia, resiliencia).

### Riesgos actuales

- Riesgo de latencia alta en redes moviles por payload excesivo.
- Riesgo de dependencia directa de microservicios sin protecciones.

### Plan de mejoras

- P0: Establecer contrato de payload minimo (campos obligatorios y maximos).
- P0: Configurar timeout/retry diferenciados para operaciones de lectura y escritura.
- P1: Compresion selectiva y versionado de payload para clientes legacy.
- P1: Metricas por tipo de dispositivo/version app.
- P2: Estrategia de fallback por funcionalidad (degradar parcialmente, no fail total).

### KPI objetivo

- P95 latencia endpoints moviles <= 300 ms.
- Tamaño medio de payload <= 50 KB.
- Error funcional en cliente por backend <= 0.5%.

### Criterio de exito

- Experiencia movil estable en condiciones de red variable.

---

## microservice-users

### Rol principal

- Identidad, perfil, roles administrativos y eventos de gameplay de usuario.

### Riesgos actuales

- Riesgo de inconsistencia de roles/permisos.
- Riesgo de trazabilidad incompleta de cambios administrativos.

### Plan de mejoras

- P0: Modelo de autorizacion explicito por endpoint (RBAC con matriz auditable).
- P0: Audit log obligatorio para cambios de rol y operaciones admin.
- P1: Idempotencia en registro de eventos de juego.
- P1: Validacion de schema en entrada y salida con contratos compartidos.
- P2: Reportes de calidad de datos (duplicados, huérfanos, campos faltantes).

### KPI objetivo

- Cambios de rol sin audit log = 0.
- Eventos duplicados < 0.1%.
- Errores de autorizacion inesperada < 0.2%.

### Criterio de exito

- Control de acceso confiable y trazable extremo a extremo.

---

## microservice-quizz

### Rol principal

- Generacion y persistencia de contenido quiz con control de calidad.

### Riesgos actuales

- Fallos de integracion con AI (403) y bajo rendimiento de generacion efectiva.
- Alto costo de intentos sin conversion en contenido util.

### Plan de mejoras

- P0: Verificacion de credenciales AI al arranque (smoke obligatorio a `catalogs` y `generate`).
- P0: Corta circuito tras N fallos 4xx/5xx consecutivos para evitar gasto inutil.
- P0: Registrar taxonomia de fallo (`auth_error`, `validation_error`, `upstream_timeout`, `duplicate`).
- P1: Reintentos con backoff solo en errores transitorios (5xx/timeouts), nunca en 4xx.
- P1: Heuristica anti-duplicados pre-insercion con score semantico configurable.
- P2: Pipeline de validacion de calidad de pregunta (completitud, dificultad coherente, lenguaje).

### KPI objetivo

- `success_ratio` generacion >= 0.8.
- `outbound_failures / outbound_requests` <= 0.05.
- Costo por item generado reducido >= 40%.

### Criterio de exito

- El batch solicitado produce contenido persistido y util de forma consistente.

---

## microservice-wordpass

### Rol principal

- Generacion y persistencia de contenido wordpass con control de duplicados.

### Riesgos actuales

- Duplicado de topico recurrente y baja conversion de intentos a contenido guardado.
- Dependencia fuerte de AI sin proteccion suficiente.

### Plan de mejoras

- P0: Mismo hardening de autenticacion y smoke de AI que quiz.
- P0: Penalizacion de topicos repetidos por ventana temporal.
- P1: Catalogo de prompts por categoria/lenguaje/dificultad con versionado.
- P1: Indicador de novedad semantica por item para acceptance/reject.
- P2: Reentrenar reglas de validacion de salida para reducir fallos no accionables.

### KPI objetivo

- `duplicate_ratio` <= 0.1.
- `failure_ratio` <= 0.2.
- `created_total / requested_total` >= 0.75.

### Criterio de exito

- Produccion sostenida de contenido no repetido y de calidad.

---

## ai-engine-api

### Rol principal

- API principal de generacion/ingesta AI con cache, RAG y control de dependencias.

### Riesgos actuales

- Riesgo de cuello de botella en llama y timeouts sin estrategia adaptativa.
- Riesgo de acoplamiento fuerte entre modo de cache y calidad de respuesta.

### Plan de mejoras

- P0: Presupuestos de timeout por etapa (`rag`, `llm`, `parse`) y fail-fast por tramo.
- P0: Limites de concurrencia por endpoint y aislamiento por tipo de workload.
- P1: Politica de cache por endpoint con hit-rate objetivo y TTL por dominio.
- P1: Exponer metrica de calidad de respuesta (validacion schema y semantic checks).
- P2: Shadow traffic para evaluar cambios de prompts/modelos sin impacto en produccion.

### KPI objetivo

- P95 latencia `generate` <= 8s (segun modelo objetivo).
- `cache_hit_rate` >= 0.35 para workloads repetibles.
- Errores de parseo <= 1%.

### Criterio de exito

- Generacion estable con calidad controlada y costos predecibles.

---

## ai-engine-stats

### Rol principal

- Observabilidad AI: ingesta de eventos, agregados operativos y metricas de cache/runtime.

### Riesgos actuales

- Riesgo de baja utilidad si productores no envian eventos de forma confiable.
- Riesgo de dashboards con metricas incompletas o poco accionables.

### Plan de mejoras

- P0: Contrato de evento obligatorio con validacion estricta y version.
- P0: Metrica de completitud de telemetria por productor.
- P1: Dashboards por flujo (`quiz`, `wordpass`, `manual`) con SLO y costo por resultado.
- P1: Alertas por degradacion de calidad (no solo por caida tecnica).
- P2: Retencion por tiers y export historico para analisis.

### KPI objetivo

- Cobertura de eventos esperados >= 95%.
- Tiempo deteccion de degradacion <= 5 min.
- MTTD + MTTR documentados por incidente.

### Criterio de exito

- La observabilidad permite explicar claramente por que falla o rinde mal un flujo AI.

---

## llama-server

### Rol principal

- Inferencia LLM base para ai-engine.

### Riesgos actuales

- Saturacion por concurrencia y contexto grande.
- Variabilidad de latencia con impacto cascada.

### Plan de mejoras

- P0: Limites de concurrencia y cola controlada con rechazo explicito.
- P0: Autoajuste de parametros por tipo de carga (ctx, batch, threads).
- P1: Warmup y health profundo (modelo cargado, memoria disponible, throughput minimo).
- P1: Benchmark periodico por modelo/version.
- P2: Plan de capacidad (CPU/GPU) con umbrales de escalado.

### KPI objetivo

- Throughput tokens/s dentro de rango objetivo por modelo.
- Timeout rate <= 1%.
- Variacion P95/P50 latencia <= 2.5x.

### Criterio de exito

- Inferencia predecible y dimensionada al workload real.

---

## Secuencia recomendada de ejecucion (servicios)

1. P0 de autenticacion, forwarding y circuit breaker en quiz/wordpass/bff-backoffice.
2. P0 de observabilidad minima estandar en todos los servicios.
3. P1 de calidad de generacion y eficiencia de costos.
4. P2 de optimizacion avanzada y escalado.

---

## Acuerdos operativos del Bloque A

### Owners tecnicos por servicio (owner y backup)

| Servicio | Owner tecnico | Backup tecnico |
|---|---|---|
| backoffice | Squad Backoffice Frontend | Squad Backoffice Backend |
| api-gateway | Squad Edge Platform | Squad SRE Platform |
| bff-backoffice | Squad Backoffice Backend | Squad Edge Platform |
| bff-mobile | Squad Mobile Backend | Squad Edge Platform |
| microservice-users | Squad Identity and Access | Squad Backoffice Backend |
| microservice-quizz | Squad Games Runtime | Squad AI Platform |
| microservice-wordpass | Squad Games Runtime | Squad AI Platform |
| ai-engine-api | Squad AI Platform | Squad AI Runtime |
| ai-engine-stats | Squad AI Observability | Squad AI Platform |
| llama-server | Squad AI Runtime | Squad SRE Platform |

### SLO y error budget acordado por servicio

Supuesto comun para presupuesto mensual: ventana de 30 dias.

| Servicio | Tipo | SLO mensual | Error budget mensual |
|---|---|---|---|
| api-gateway | edge | 99.9% disponibilidad | 43.2 min de indisponibilidad |
| bff-backoffice | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| bff-mobile | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| backoffice | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| microservice-users | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| microservice-quizz | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| microservice-wordpass | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| ai-engine-api | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| ai-engine-stats | interno | 99.5% disponibilidad | 216 min de indisponibilidad |
| llama-server | interno | 99.5% disponibilidad | 216 min de indisponibilidad |

---

## Checklist de ejecucion del Plan 1 (servicios)

### Bloque A. Fundacion operativa

- [x] Definir owner tecnico por servicio.
- [x] Publicar responsabilidad principal por servicio en su README.
- [x] Capturar baseline de KPI (latencia, error rate, disponibilidad, eficacia).
- [x] Acordar SLO y error budget por servicio.

### Bloque B. P0 de seguridad y resiliencia

- [x] Validar forwarding de headers criticos en edge y BFF.
- [x] Corregir rutas con auth incompleta (incluyendo AI keys donde aplique).
- [x] Implementar timeout estandar por tipo de llamada.
- [x] Implementar circuito de proteccion para dependencias criticas.

### Bloque C. P0 de observabilidad

- [x] Estandarizar logs JSON con correlation_id.
- [x] Exponer metricas Prometheus por endpoint clave.
- [x] Activar trazas distribuidas en hops interservicio.
- [x] Crear alertas P0 con runbook enlazado.

### Bloque D. Eficacia funcional (foco quiz y wordpass)

- [x] Smoke de credenciales AI al arranque.
- [x] Regla de no ejecutar batch si smoke falla.
- [x] Clasificar errores en taxonomia operativa.
- [x] Medir conversion real: requested -> created por batch.

### Bloque E. Cierre de iteracion

- [x] Ejecutar regresion tecnica (build + test + contrato).
- [x] Comparar KPI pre/post y documentar impacto.
- [x] Publicar acta de cierre con pendientes P1/P2.

### Criterio de salida del Plan 1

- [x] No hay errores recurrentes por headers faltantes.
- [ ] Todos los servicios exponen metricas y logs consumibles.
- [ ] Flujos criticos tienen trazabilidad end-to-end.
- [x] quiz y wordpass pasan de estado rojo a estado operativo medible.
