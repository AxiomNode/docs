# ADR 0009 - LLMOps y eval harness en ai-engine

- Status: Accepted
- Date: 2026-04-22

## Context

Los bloques 4 y 8 del *Máster en Desarrollo con IA* exigen tratar a los sistemas basados en LLMs como sistemas operativos de pleno derecho: con métricas de calidad de respuesta, observabilidad específica, control de coste y versionado de prompts y modelos. Esto es lo que el máster denomina **LLMOps**.

`ai-engine` ya está separado en generación, stats, caché y runtime llama (ADR 0004), pero hoy:

- No existe un *eval harness* que ejecute un dataset de referencia y mida regresiones de calidad entre versiones.
- No se miden latencia por modelo, tasa de fallback, coste por token ni error rate por prompt como métricas de primera clase.
- Los prompts de sistema viven en código y cambian sin trazabilidad explícita.
- El llama runtime opcional puede sustituirse o degradarse sin que ningún test lo detecte automáticamente.

Sin LLMOps básico, cualquier ajuste en prompts, modelos o infraestructura puede degradar la calidad de la experiencia sin alarma.

## Decision

Adoptar LLMOps como práctica formal en `ai-engine` con los siguientes compromisos:

1. **Eval harness mínimo en `ai-engine`**:
   - Dataset de prompts de referencia bajo `ai-engine/src/tests/eval/` con expected behaviors (no necesariamente expected outputs literales).
   - Job ejecutable localmente y en CI mediante `make eval` o equivalente.
   - Métricas de salida: tasa de éxito por categoría, longitud promedio, latencia p50/p95, coste/llamada.
2. **Versionado de prompts**:
   - Los prompts de sistema viven en archivos versionados bajo `ai-engine/src/ai_engine/prompts/` con cabecera `version: vN`.
   - Cualquier cambio de prompt incrementa la versión y deja entry en CHANGELOG del propio repo.
3. **Métricas de runtime LLM**:
   - `ai-engine` exporta métricas Prometheus específicas: `ai_engine_llm_latency_seconds`, `ai_engine_llm_fallback_total`, `ai_engine_llm_tokens_total{direction="in|out"}`, `ai_engine_llm_errors_total{kind}`.
   - `observability-platform` añade dashboard "AI engine LLMOps" en `observability-platform/dashboards/`.
4. **Política de fallback explícita**:
   - Si el llama runtime no responde en N segundos, `ai-engine` degrada a respuesta canned o a modelo alternativo y emite métrica.
   - Documentado en `ai-engine/README.md` y en `docs/operations/ai-services-recovery-runbook.md`.
5. **Gate de regresión en CI**:
   - El eval harness corre como *job no bloqueante* primero (warning); tras una iteración estabilizadora pasa a bloqueante con umbrales acordados.

## Alternatives considered

1. Esperar a tener volumen real de tráfico antes de invertir en LLMOps: rechazado; cuando llega el volumen, las regresiones ya han ocurrido.
2. Usar una plataforma externa de LLMOps (LangSmith, Helicone, Phoenix): viable y compatible; el ADR fija las prácticas, no el proveedor. Se podrá superpuestar más adelante.
3. Mantener únicamente logs y revisar manualmente: rechazado; no detecta regresión de calidad.

## Consequences

- Esfuerzo inicial para construir el dataset de referencia y la infraestructura de métricas.
- Pipeline de `ai-engine` algo más lento por la fase eval.
- Trazabilidad real de calidad de generación a lo largo del tiempo.
- Base sólida para introducir RAG o fine-tuning en el futuro sin perder visibilidad.

## Related

- [ADR 0004 - Split AI runtime](./0004-split-ai-runtime.md)
- [Master alignment](../master-alignment.md) - pilares 4 y 8
- [AI services recovery runbook](../../operations/ai-services-recovery-runbook.md)
- [Quality attribute scenarios](../quality-attributes.md)
