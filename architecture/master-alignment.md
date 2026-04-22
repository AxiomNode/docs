# Alineación con el Máster en Desarrollo con IA

Last updated: 2026-04-23.

Este documento mapea los pilares del *Máster en Desarrollo con IA* (resumido en `.tmp/MásterDesarrolloIA/summary/`) con el estado actual de la plataforma AxiomNode. Para cada pilar se describe qué se cumple hoy, qué brechas existen y qué acciones concretas se proponen, enlazando a los artefactos donde se materializan.

El objetivo es doble: (1) usar el máster como vara de medir externa para detectar deuda de ingeniería, arquitectura, calidad y seguridad; y (2) dejar trazabilidad explícita entre el conocimiento académico del propietario y las decisiones técnicas del proyecto.

## Resumen ejecutivo

| Pilar del máster | Estado AxiomNode | Brecha principal | Acción propuesta |
|---|---|---|---|
| 2. Ingeniería de software | Cubierto: ciclo de vida, requisitos como casos de uso, principios de diseño aplicados por repo | No hay un mapa de historias de usuario formal por canal | Añadir story map por canal en `docs/architecture/use-cases.md` |
| 3. Arquitectura de software | Cubierto: C4 L1-L3, dominio, ADRs 0001-0007, monolito modular + microservicios + event-driven parcial | Falta formalizar Outbox y la matriz síncrono/asíncrono entre servicios | Documentar Outbox y comunicación inter-servicios; ADR si hay decisión nueva |
| 4. Fundamentos de IA | Cubierto: split AI runtime (ADR 0004), generación + stats + cache + llama, eval harness baseline + prompts versionados | Faltan métricas de calidad sobre LLM real y documento de IA responsable | Promover eval a real-LLM gate; redactar `responsible-ai.md` |
| 6. Flujo de desarrollo con IA | Parcial: Copilot en uso, prompts ad-hoc | No hay base de conocimiento ni reglas de prompting versionadas | Añadir `docs/guides/ai-prompting-rules.md` |
| 7. Calidad | Cubierto: vitest 90%, pytest 90%, Codecov en 8 repos, observabilidad mínima | Falta E2E de extremo a extremo y métricas DORA | Añadir Playwright E2E al menos para backoffice; medir DORA |
| 8. Infra y Cloud | Cubierto: Docker, K8s, CI/CD, tags inmutables (ADR 0007), platform-infra centralizado (ADR 0006) | Falta LLMOps formal | ADR 0009 |
| 9. Seguridad | Parcial: secretos centralizados, SAST (CodeQL) + SCA (Trivy/`npm audit`/`pip-audit`) en CI de los 8 repos runtime | Falta threat model, prompt-injection policy y rotación de secretos | Threat model STRIDE + endurecimiento prompt-injection + runbook rotación |
| 11. Productividad | Cubierto: governance docs, templates, plantillas PR/issue/discussion | Falta retrospectiva técnica periódica | Añadir runbook de retros trimestrales |

## Pilares y mapeo detallado

### 1. Preparación y mentalidad (Bloque 1 del máster)

**Master:** mentalidad de aprendizaje continuo, entorno listo, IA como apoyo, no como muleta.

**Estado AxiomNode:**

- Repos con `AGENTS.md` que codifican cómo deben colaborar agentes IA y humanos.
- `CONTRIBUTING.md` y plantillas de PR/Issue/Discussion ya creadas en governance baseline.

**Brechas:** ninguna estructural. La filosofía de uso ya está implícita en `AGENTS.md`.

**Acción:** ninguna nueva en esta iteración.

### 2. Ingeniería de software (Bloque 2)

**Master:** ciclo de vida completo, análisis de requisitos con UML/historias, paradigmas, principios SOLID/DRY/KISS/YAGNI.

**Estado AxiomNode:**

- [docs/architecture/use-cases.md](./use-cases.md) recoge 16 casos de uso con actores y flujos.
- Cada repo aplica principios de diseño en su capa (Clean-ish en backoffice, hexagonal parcial en microservicios).

**Brechas:**

- No existe un *story map* por canal (mobile vs backoffice) que vincule épicas con casos de uso.
- Los criterios de aceptación viven en issues, no en docs estables.

**Acción propuesta:**

- Extender `use-cases.md` con un story map por canal en una iteración futura.
- Mantener criterios de aceptación dentro de templates de issue (ya hechos).

### 3. Arquitectura de software (Bloque 3)

**Master:** Clean Architecture, monolito modular vs microservicios, comunicación síncrona/asíncrona, EDA, patrón Outbox.

**Estado AxiomNode:**

- C4 L1, L2, L3 documentados en [c4-views.md](./c4-views.md).
- Dominio modelado en [domain-model.md](./domain-model.md) con bounded contexts.
- 7 ADRs cubren edge único, BFF por canal, contratos como SoT, split AI, runtime control plane, infra centralizada, tags inmutables.
- Comunicación principalmente síncrona REST. Hay eventos puntuales en `ai-engine` y `microservice-quizz`, no formalizados.

**Brechas:**

- No hay un documento de comunicación inter-servicios que enumere qué llamadas son síncronas vs asíncronas.
- No se aplica patrón Outbox de forma sistemática para garantizar consistencia eventual.

**Acción propuesta:**

- Crear `docs/architecture/inter-service-communication.md` (iteración futura) listando contratos síncronos y eventos.
- Si se adopta Outbox, redactar ADR 0010.

### 4. Fundamentos de IA (Bloque 4)

**Master:** LLMs, prompting, RAG, fine-tuning, agents, IA responsable.

**Estado AxiomNode:**

- `ai-engine` ya separa generación, stats, caché y runtime llama (ADR 0004).
- Prompts y configuraciones viven en `ai-engine/src/ai_engine/`.

**Brechas:**

- No hay métricas de calidad sobre LLM real (el harness actual usa respuestas canned).
- No hay métricas de RAG (si se añade RAG en el futuro).
- No hay documento de *IA responsable* con políticas de privacidad de prompts y datos de usuario.

**Avances recientes (2026-04-23):**

- Eval harness baseline en `ai-engine/src/tests/eval/` con dataset YAML, runner, reporte JSON y test pytest (`@pytest.mark.eval`).
- Job CI `eval` no bloqueante añadido en `ai-engine/.github/workflows/ci.yml` con upload del reporte como artifact.
- Versionado explícito de prompts en `ai_engine/games/prompts.py` (`PROMPT_VERSIONS`, `get_prompt_version`).

**Acción propuesta:**

- Ver ADR 0009 (LLMOps y eval harness) más abajo.
- Añadir `docs/architecture/responsible-ai.md` cuando se introduzca RAG o fine-tuning.

### 5. Herramientas (Bloque 5)

**Master:** VS Code + Copilot, JetBrains, Cursor, Warp, CLIs de IA, n8n, CodeRabbit.

**Estado AxiomNode:**

- VS Code + Copilot es el flujo principal del propietario.
- No se usan herramientas externas como CodeRabbit en PRs.

**Brechas:**

- Code review asistido por IA no está habilitado en GitHub.

**Acción propuesta (opcional):** evaluar habilitar CodeRabbit o equivalente en PRs de los 8 repos con coverage. Sin ADR; es decisión operativa.

### 6. Flujo de desarrollo con IA (Bloque 6)

**Master:** prompt engineering, reglas, bases de conocimiento, integración con APIs IA.

**Estado AxiomNode:**

- `AGENTS.md` por repo define el contrato con agentes IA.
- No hay un repositorio centralizado de prompts ni reglas de prompt engineering.

**Brechas:**

- Prompts del backoffice y de `ai-engine` no están versionados como artefactos de primera clase.
- No hay base de conocimiento RAG sobre la propia documentación de AxiomNode.

**Acción propuesta:**

- Añadir `docs/guides/ai-prompting-rules.md` con plantillas reutilizables (futura iteración).
- Considerar indexar `docs/` con un asistente RAG interno (proyecto P3 del máster).

### 7. Calidad (Bloque 7)

**Master:** mapa de pruebas, TDD con IA, refactor seguro, métricas honestas, observabilidad (Sentry), seguridad ENV+OWASP, Docs as Code, ADRs, usabilidad.

**Estado AxiomNode:**

- Vitest con umbral 90% en 6 repos Node y pytest con `--cov-fail-under=90` en `ai-engine`.
- Codecov ya integrado en 8 repos con badges y CI.
- Observabilidad mínima vía `observability-platform` (Prometheus + Grafana stack).
- ADRs y Docs as Code ya activos.

**Brechas:**

- Pruebas E2E reales: ausentes. No hay Playwright contra el backoffice ni contra el flujo `mobile-app -> api-gateway -> microservicios`.
- Métricas DORA (lead time, deploy frequency, MTTR, change failure rate): no se miden.
- Sentry: no integrado; sólo hay logs estructurados.
- Heurísticas de usabilidad y accesibilidad WCAG: no auditadas.

**Acción propuesta:**

- Añadir Playwright al backoffice como mínimo (suite de smoke E2E).
- Empezar a recolectar DORA usando los datos de GitHub Actions y los rollouts de `platform-infra`.
- Evaluar Sentry o GlitchTip self-hosted en `observability-platform`.

### 8. Infraestructura y cloud (Bloque 8)

**Master:** DevOps, CI/CD, cloud, contenedores, LLMOps.

**Estado AxiomNode:**

- CI/CD completo por repo con `validate-infra`, `validate-contracts`, build → push → deploy controlado.
- Tags inmutables (ADR 0007).
- Despliegue en VPS para servicios y local para `ai-engine` en dev.
- LLMOps: no formalizado.

**Brechas:**

- No hay observabilidad específica de LLMs (latencia por modelo, error rate por prompt, coste por token, tasa de fallback).
- No hay versionado explícito de prompts ni de modelos cargados en el llama runtime.

**Acción propuesta:**

- Ver ADR 0009 (LLMOps).

**Avances recientes (2026-04-23):**

- Métricas LLMOps específicas expuestas vía `/metrics` en `ai-engine`: `ai_engine_llm_latency_seconds`, `ai_engine_llm_latency_p95_seconds`, `ai_engine_llm_fallback_total`, `ai_engine_llm_tokens_total{direction}`, `ai_engine_llm_errors_total{kind}`.
- Dashboard Grafana `ai-engine-llmops.json` añadido en `observability-platform/dashboards/` con panels para latencia avg/p95, fallback total, tokens in/out, errores y success/retry rate.

### 9. Seguridad (Bloque 9)

**Master:** security by design, OWASP Top 10, SSDLC, DevSecOps, prácticas de codificación segura, IA en seguridad y nuevos riesgos (prompt injection).

**Estado AxiomNode:**

- Secretos centralizados en `secrets/` con guía de distribución.
- Pod security baseline en K8s.
- npm audit periódico; pip-audit ad-hoc.
- Branch protection y validaciones de PR activas.

**Brechas:**

- No hay un *threat model* (STRIDE/LINDDUN) por servicio.
- No hay políticas explícitas contra prompt injection en `ai-engine` y BFFs.
- No hay rotación documentada de secretos.

**Avances recientes (2026-04-23):**

- SAST con CodeQL v4 y SCA con Trivy fs (HIGH/CRITICAL bloqueante) integrados como workflow `security.yml` en los 8 repos runtime.
- `ai-engine` añade además `pip-audit` sobre `requirements.txt`.
- `npm audit --omit=dev --audit-level=high` ya estaba en los 7 repos Node.
- Política contra prompt injection formalizada en [`docs/guides/ai-prompting-rules.md`](../guides/ai-prompting-rules.md), con reglas duras para `ai-engine` y BFFs y referencias a las métricas LLMOps de ADR 0009.
- Procedimiento de rotación de secretos publicado en [`docs/operations/secrets-rotation-policy.md`](../operations/secrets-rotation-policy.md) (cadencias, roles, rotación de emergencia y auditoría continua).
- Reglas duras de comunicación entre servicios consolidadas en [`docs/guides/inter-service-communication.md`](../guides/inter-service-communication.md), apoyadas en `service-communication.md`.
- Proceso de retros y post-mortems documentado en [`docs/operations/retrospective-process.md`](../operations/retrospective-process.md).

**Acción propuesta:**

- Ver ADR 0008 (DevSecOps + SSDLC) más abajo.
- Añadir `docs/operations/threat-model.md` cuando se ejecute el primer ejercicio.

### 10. Desarrollo potenciado por IA (Bloque 10)

**Master:** proyectos integrales front, mobile, backend, IA.

**Estado AxiomNode:** AxiomNode es exactamente eso. Cubre los cuatro mundos en una sola plataforma multi-repo.

**Brechas:** ninguna conceptual. Las brechas se derivan de los pilares anteriores.

### 11. Productividad y estrategia (Bloque 11)

**Master:** ágil, backlog, retros, soft skills, freelancing.

**Estado AxiomNode:**

- Templates PR/Issue/Discussion publicados.
- Convenciones de commit y de governance documentadas.

**Brechas:**

- No hay cadencia documentada de retrospectivas técnicas.

**Acción propuesta:** añadir `docs/operations/tech-retrospective-cadence.md` (iteración futura, no bloqueante).

## Decisiones que esta alineación introduce

Como resultado directo de este mapeo se redactan dos ADRs nuevos:

- [ADR 0008 - DevSecOps y SSDLC](./decision-records/0008-devsecops-and-ssdlc.md)
- [ADR 0009 - LLMOps y eval harness en ai-engine](./decision-records/0009-llmops-and-ai-evaluation.md)

Las acciones marcadas como "iteración futura" no se materializan en este commit; quedan registradas aquí como deuda visible y se priorizarán en próximos ciclos.

## Cómo mantener este documento

- Revisar cada vez que se complete un nuevo ADR o se cierre una brecha listada.
- Si una brecha desaparece, mover su fila a la sección de pilares como "Cumplido" y eliminar la fila del resumen ejecutivo.
- No usar este documento para roadmap aspiracional; sólo para fotografía actual + decisiones tomadas.

## Referencias

- Resumen del máster: `.tmp/MásterDesarrolloIA/summary/overview.md`
- Plan de implementación del máster: `.tmp/MásterDesarrolloIA/summary/HowToImplement.md`
- Aprendizajes consolidados: `.tmp/MásterDesarrolloIA/summary/Learnings.md`
