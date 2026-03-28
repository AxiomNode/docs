# 02. Planes Por Repositorio (Pendientes Vigentes)

Fecha de actualizacion: 2026-03-28.

Este tablero conserva solo trabajo pendiente. Todo item ya ejecutado en iteraciones previas fue retirado.

## Pendientes P0

| Repositorio | Pendiente | Owner propuesto | KPI de salida |
|---|---|---|---|
| ai-engine | Pruebas de contrato por endpoint para `X-API-Key` (matriz de aceptacion/rechazo por ruta) | Squad AI Platform | Errores de auth por contrato = 0 en suite de regresion |
| bff-backoffice | Contrato de agregacion estable y versionado para endpoints de observabilidad/data | Squad Backoffice Backend | Incidentes por cambio de contrato = 0 |
| bff-mobile | Contratos de payload minimos por version de app publicados y validados | Squad Mobile Backend | Error rate por version dentro de umbral objetivo |
| backoffice | Pruebas e2e de rutas operativas criticas | Squad Backoffice Frontend | Flujos criticos e2e en verde en CI |
| microservice-users | Fortalecer RBAC y auditoria de roles | Squad Identity and Access | Conflictos de rol/evento por 10k dentro de objetivo |
| microservice-wordpass | Politicas anti-duplicado de topico y contenido | Squad Games Runtime | Ratio de duplicados por batch bajo umbral |
| contracts-and-schemas | Contract tests automaticos en productores y consumidores | Squad Contracts Platform | Incidentes por incompatibilidad de contrato = 0 |
| shared-sdk-client | Regeneracion automatica por cambio de contrato | Squad Developer Experience | Fallos de integracion por desalineacion SDK/contrato = 0 |
| platform-infra | Plantillas estandar de variables + validacion pre-deploy | Squad SRE Platform | Fallos de despliegue por config/secrets en descenso sostenido |
| platform-infra | Secret management centralizado y rotacion automatizada | Squad SRE Platform | Incidentes por credenciales fuera de politica = 0 |
| observability-platform | Dashboard unico de SLO por servicio | Squad AI Observability | Visibilidad SLO completa para servicios criticos |
| docs | Ownership por documento + fecha de ultima validacion | Squad Docs Enablement | % docs vigentes en 90 dias por encima de objetivo |
| secrets | Eliminar uso de secretos fuera de vault autorizado | Squad Security Platform | Incidentes por fuga de secreto = 0 |
| secrets | Rotacion trimestral y revocacion por incidente | Squad Security Platform | Tiempo de revocacion dentro de SLA |

## Pendientes P1

| Repositorio | Pendiente | Owner propuesto | KPI de salida |
|---|---|---|---|
| ai-engine | Benchmarks reproducibles por modelo/distribucion | Squad AI Runtime | Latencia/calidad comparables por release |
| ai-engine | Tooling de comparacion de prompts/versiones en shadow | Squad AI Runtime | Desviaciones detectadas antes de promocion |
| api-gateway | Limites por consumidor y anti-abuso | Squad Edge Platform | Caidas por abuso reducidas |
| api-gateway | Plantillas de error consistentes por dominio | Squad Edge Platform | Errores homogenizados en rutas edge |
| bff-backoffice | Cache/coalescing para requests repetidas de panel | Squad Backoffice Backend | Menor latencia y menor carga aguas abajo |
| bff-backoffice | Endpoint de estado agregado con puntuacion de salud | Squad Backoffice Backend | Diagnostico operativo mas rapido |
| bff-mobile | Telemetria por version de app y calidad de red | Squad Mobile Backend | Visibilidad por cohortes de red/version |
| bff-mobile | Versionado backward compatible con sunset plan | Squad Mobile Backend | Incidentes por compatibilidad en descenso |
| backoffice | Registro de auditoria de operador | Squad Backoffice Frontend | Trazabilidad de acciones administrativas completa |
| backoffice | Modo degradado por dependencia | Squad Backoffice Frontend | Disponibilidad funcional durante degradacion |
| microservice-users | Validadores de calidad de datos de perfil/evento | Squad Identity and Access | Rechazos de datos inconsistentes controlados |
| microservice-users | Reportes de consistencia y reconciliacion | Squad Identity and Access | Hallazgos de inconsistencias detectados temprano |
| microservice-quizz | Reintentos inteligentes para errores transitorios | Squad Games Runtime | Mejora de `created/requested` sin aumentar fallos permanentes |
| microservice-quizz | Observabilidad de costo por item creado | Squad Games Runtime | Costo por item visible por ventana temporal |
| microservice-wordpass | Libreria compartida de validacion de salida | Squad Games Runtime | Menos rechazos por payload invalido |
| microservice-wordpass | Telemetria de novedad/variedad de contenido | Squad Games Runtime | Variabilidad de contenido dentro de objetivo |
| contracts-and-schemas | Ejemplos positivos/negativos por endpoint/evento | Squad Contracts Platform | Cobertura de ejemplos para contratos criticos |
| contracts-and-schemas | Diff semantico de breaking changes en CI | Squad Contracts Platform | Bloqueo automatico de cambios incompatibles |
| shared-sdk-client | Version pinning y matriz de compatibilidad | Squad Developer Experience | Compatibilidad documentada por version |
| shared-sdk-client | Pruebas de backward compatibility | Squad Developer Experience | Regresiones de SDK detectadas en CI |
| platform-infra | Health checks de arranque dependiente por stack | Squad SRE Platform | Menos fallos al iniciar stacks locales |
| platform-infra | Rollback automatico ante SLO breach inicial post-deploy | Squad SRE Platform | MTTR post-deploy reducido |
| observability-platform | Mapa de dependencias en tiempo real | Squad AI Observability | Impacto de fallos visible por cadena |
| observability-platform | Indicadores de negocio ademas de tecnicos | Squad AI Observability | Correlacion tecnica-negocio visible |
| docs | Plantillas para ADR, incidentes y postmortems | Squad Docs Enablement | Documentos consistentes y auditables |
| docs | Docs as code con validacion de enlaces y ejemplos | Squad Docs Enablement | Errores de docs detectados en CI |
| secrets | Scanners de secretos en pre-commit y CI | Squad Security Platform | Hallazgos de secretos en PR en descenso |

## Pendientes P2

| Repositorio | Pendiente | Owner propuesto | KPI de salida |
|---|---|---|---|
| ai-engine | Gobernanza de modelos (promocion y rollback) | Squad AI Platform | Cambios de modelo trazables y reversibles |
| api-gateway | Estrategia de despliegue canary por ruta | Squad Edge Platform | Riesgo de despliegue reducido |
| bff-backoffice | Simulador de incidentes para validar UX operacional | Squad Backoffice Backend | Mejor tiempo de respuesta en drills |
| bff-mobile | Flags de feature para cambios progresivos | Squad Mobile Backend | Rollout controlado por version/cohorte |
| backoffice | Dashboard de productividad operativa | Squad Backoffice Frontend | Mejoras medibles en tiempo de resolucion |
| microservice-users | Hardening de privacidad y minimizacion de datos | Squad Identity and Access | Menor exposicion de datos sensibles |
| microservice-quizz | Control de calidad semantica de contenido | Squad Games Runtime | Menos contenido invalido en produccion |
| microservice-wordpass | Ajuste dinamico de prompts por resultados | Squad Games Runtime | Mejora sostenida de conversion |
| contracts-and-schemas | Catalogo de errores estandar | Squad Contracts Platform | Taxonomia de errores reutilizable |
| shared-sdk-client | Plantillas de retries/timeouts por lenguaje | Squad Developer Experience | Integraciones mas robustas entre SDKs |
| platform-infra | Cost observability por servicio/entorno | Squad SRE Platform | Costo por servicio monitoreado |
| observability-platform | Deteccion de anomalias basada en baseline | Squad AI Observability | MTTD reducido en incidentes anomalo |
| docs | Indices por rol (dev, ops, product) | Squad Docs Enablement | Descubrimiento de informacion mas rapido |
| secrets | Reporte de exposicion y cumplimiento | Squad Security Platform | Auditoria de cumplimiento periodica cerrada |

## Checklist operativo global

- [ ] DoD actualizado y aprobado por repositorio.
- [ ] Pipeline CI con gates activos por repositorio.
- [ ] Contratos versionados y validados en rutas criticas.
- [ ] Runbook operativo disponible para cada servicio runtime.
