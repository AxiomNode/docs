# 02. Planes Por Repositorio

## Criterios transversales para todos los repositorios

- Definition of Done obligatoria:
  - tests unitarios/integracion,
  - metricas instrumentadas,
  - logs estructurados,
  - contrato actualizado,
  - runbook operativo.
- Pipeline CI con quality gates:
  - cobertura,
  - lint,
  - type-check,
  - pruebas de contrato.
- Politica de versionado semantico y changelog por repo.

---

## ai-engine

### Responsabilidad

- Motor AI y observabilidad AI.

### Mejoras recomendadas

- P0: Separar claramente dominios `ai-api` y `ai-stats` (codigo, config, ownership).
- P0: Pruebas de contrato para headers de auth (`X-API-Key`) por endpoint.
- P1: Benchmarks reproducibles por modelo y distribucion.
- P1: Tooling para comparar prompts/versiones en modo shadow.
- P2: Politica de gobernanza de modelos (promocion y rollback).

### Medicion clave

- Regression suite de latencia/calidad por release.

---

## api-gateway

### Responsabilidad

- Edge routing y politicas transversales.

### Mejoras recomendadas

- P1: Limites por consumidor y proteccion anti-abuso.
- P1: Plantillas de error consistentes por dominio.
- P2: Estrategia de despliegue canary por ruta.

### Medicion clave

- Tasa de errores por upstream y por ruta.

---

## bff-backoffice

### Responsabilidad

- Orquestacion de datos operativos para backoffice.

### Mejoras recomendadas

- P0: Contrato de agregacion estable y versionado.
- P0: Fallback por dependencia con degradacion explicita.
- P1: Cache y coalescing de requests repetidas del panel.
- P1: Endpoint de estado agregado con puntuacion de salud.
- P2: Simulador de incidentes para validar UX operacional.

### Medicion clave

- Porcentaje de respuestas parciales vs fallos totales.

---

## bff-mobile

### Responsabilidad

- API adaptada a cliente movil.

### Mejoras recomendadas

- P0: Contratos de payload minimos por version de app.
- P0: Timeouts y retries orientados a red movil.
- P1: Telemetria por version de app y calidad de red.
- P1: Versionado backward compatible con sunset plan.
- P2: Flags de feature para cambios progresivos.

### Medicion clave

- P95 por endpoint y error rate por version de app.

---

## backoffice

### Responsabilidad

- Operacion, diagnostico y acciones administrativas.

### Mejoras recomendadas

- P0: Estandar de estados visuales y acciones seguras.
- P0: Pruebas e2e de rutas operativas criticas.
- P1: Registro de auditoria de operador.
- P1: Modo degradado por dependencia.
- P2: Dashboard de productividad operativa.

### Medicion clave

- Tiempo medio de resolucion asistida por UI.

---

## microservice-users

### Responsabilidad

- Identidad y dominio de usuario.

### Mejoras recomendadas

- P0: Fortalecer RBAC y auditoria de roles.
- P0: Idempotencia en eventos de gameplay.
- P1: Validadores de calidad de datos de perfil/evento.
- P1: Reportes de consistencia y reconciliacion.
- P2: Hardening de privacidad y minimizacion de datos.

### Medicion clave

- Tasa de conflictos de rol/evento por 10k operaciones.

---

## microservice-quizz

### Responsabilidad

- Generacion quiz y persistencia.

### Mejoras recomendadas

- P1: Reintentos inteligentes solo para errores transitorios.
- P1: Observabilidad de costo por item creado.
- P2: Control de calidad semantica de contenido.

### Medicion clave

- `created/requested`, `failure_ratio`, costo por item.

---

## microservice-wordpass

### Responsabilidad

- Generacion wordpass y persistencia.

### Mejoras recomendadas

- P0: Politicas anti-duplicado de topico y contenido.
- P1: Libreria compartida de validacion de salida.
- P1: Telemetria de novedad/variedad de contenido.
- P2: Ajuste dinamico de prompts por resultados.

### Medicion clave

- Duplicados por batch y conversion de intentos.

---

## contracts-and-schemas

### Responsabilidad

- Fuente unica de contratos de API/eventos.

### Mejoras recomendadas

- P0: Contract tests automáticos en consumidores y productores.
- P0: Regla de no-deploy si cambia contrato sin versionado.
- P1: Ejemplos positivos/negativos por endpoint/evento.
- P1: Diff semantico de breaking changes en CI.
- P2: Catalogo de errores estandar.

### Medicion clave

- Incidentes por incompatibilidad de contrato = 0.

---

## shared-sdk-client

### Responsabilidad

- SDKs cliente para consumir contratos de forma segura.

### Mejoras recomendadas

- P0: Regeneracion automatica por cambio de contrato.
- P0: Version pinning y matriz de compatibilidad.
- P1: Telemetria opcional de uso SDK (sin PII).
- P1: Pruebas de backward compatibility.
- P2: Plantillas de retries/timeouts por lenguaje.

### Medicion clave

- Fallos de integracion por desalineacion SDK/contrato.

---

## platform-infra

### Responsabilidad

- Provisioning, configuracion de entornos y despliegue.

### Mejoras recomendadas

- P0: Plantillas estandar de variables y validacion pre-deploy.
- P0: Secret management centralizado y rotacion automatizada.
- P1: Health checks de arranque dependiente por stack.
- P1: Rollback automatizado ante SLO breach inicial post-deploy.
- P2: Cost observability por servicio/entorno.

### Medicion clave

- Fallos de despliegue por config/secrets.

---

## observability-platform

### Responsabilidad

- Dashboards, alertas y trazabilidad unificada.

### Mejoras recomendadas

- P0: Dashboard unico de SLO por servicio.
- P1: Mapa de dependencias con estado en tiempo real.
- P1: Indicadores de negocio ademas de tecnicos.
- P2: Deteccion de anomalías basada en baseline.

### Medicion clave

- MTTD y MTTR por tipo de incidente.

---

## docs

### Responsabilidad

- Documentacion operativa y tecnica viva.

### Mejoras recomendadas

- P0: Estructura uniforme (arquitectura, guias, operaciones, runbooks).
- P0: Ownership por documento y fecha de ultima validacion.
- P1: Plantillas para ADR, incidentes y postmortems.
- P1: "Docs as code" con validaciones de enlaces y ejemplos.
- P2: Indices por rol (dev, ops, product).

### Medicion clave

- % documentos vigentes y validados en los ultimos 90 dias.

---

## secrets

### Responsabilidad

- Gestion segura de credenciales y politicas.

### Mejoras recomendadas

- P0: No usar secretos en repos locales fuera de vault autorizado.
- P0: Rotacion trimestral y revocacion por incidente.
- P1: Scanners de secretos en pre-commit y CI.
- P2: Reporte de exposicion y cumplimiento.

### Medicion clave

- Incidentes por fuga de secreto = 0.

---

## Priorizacion temporal sugerida (repos)

1. Semana 1-2: P0 de contratos/auth/observabilidad basica.
2. Semana 3-4: P1 de rendimiento, cache y calidad de datos.
3. Semana 5-6: P2 de optimizacion avanzada y costos.

---

## Checklist de ejecucion por repositorio

### Base comun

- [ ] DoD actualizado y aprobado.
- [ ] Pipeline CI con gates activos.
- [ ] Contratos versionados y validados.
- [ ] Runbook operativo disponible.

### Repos criticos de runtime

- [ ] ai-engine: auth por endpoint y benchmark base.
- [ ] api-gateway: forwarding unificado y test matrix completa.
- [ ] bff-backoffice: degradacion parcial implementada.
- [ ] bff-mobile: contratos de payload por version.
- [ ] microservice-quizz: smoke AI + circuit breaker.
- [ ] microservice-wordpass: anti-duplicado + smoke AI.
- [ ] microservice-users: RBAC y audit log obligatorios.

### Soporte y plataforma

- [ ] contracts-and-schemas: contract tests automáticos activos.
- [ ] shared-sdk-client: regeneracion automatica y compatibilidad.
- [ ] platform-infra: validacion pre-deploy y secretos centralizados.
- [ ] observability-platform: dashboards SLO y alertas accionables.
- [ ] docs: ownership y vigencia de documentos.
- [ ] secrets: rotacion y scanning automatizado.
