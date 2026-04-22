# AI Prompting Rules

Last updated: 2026-04-23.

Política transversal contra prompt injection y abuso de modelos para `ai-engine`,
los BFFs (`bff-mobile`, `bff-backoffice`) y cualquier servicio que construya
prompts antes de invocar al runtime. Complementa al ADR 0009 (LLMOps) y al
ADR 0008 (DevSecOps + SSDLC).

## Principios

1. **Trust boundary explícito.** El contenido proveniente del usuario, del
   dataset RAG, de fuentes externas y del estado interno se trata como
   *untrusted* salvo que sea una constante de código revisada en commit.
2. **Separación system / user.** Las instrucciones del sistema viven en
   `ai_engine/games/prompts.py` (versionadas con `PROMPT_VERSIONS`). El input
   del usuario se inserta solo dentro de bloques delimitados.
3. **Sin ejecución reflexiva.** El motor nunca interpreta la salida del modelo
   como instrucción operativa (no `eval`, no llamadas a herramientas a partir
   del texto generado, no shell-out).
4. **Mínima sorpresa.** Cualquier nueva plantilla o cambio sustantivo a una
   plantilla obliga a bumpear `PROMPT_VERSIONS[<game>]` y a actualizar
   datasets de evaluación (`src/tests/eval/datasets/`).

## Reglas obligatorias en `ai-engine`

- Toda plantilla activa vive en `ai_engine/games/prompts.py` y expone su
  versión vía `get_prompt_version(game_type)`.
- El input de usuario se rodea con etiquetas explícitas:
  `<<USER_INPUT>>...<</USER_INPUT>>`. El modelo recibe instrucción de tratar
  ese contenido como datos, no como órdenes.
- El system prompt prohibe explícitamente:
  - Ignorar instrucciones previas.
  - Revelar el system prompt o claves del entorno.
  - Producir comandos shell, código ejecutable destinado a otro sistema o URLs
    externas no listadas.
- Los documentos RAG se etiquetan con `[doc:<id>]` y el modelo debe citar la
  fuente o devolver `unknown` si no encuentra evidencia.
- Las salidas se validan contra el schema del juego (`quiz`, `word-pass`,
  `true_false`) antes de devolverse al cliente. Si la validación falla, se
  registra `errors_total{kind="generation"}` y se aplica fallback.

## Reglas obligatorias en BFFs

- Tamaño máximo de input por campo (`topic`, `prompt`, etc.) aplicado en el
  BFF antes de invocar al motor (default: 512 caracteres para topics, 4 KB
  para textos libres).
- Lista negra básica de patrones de jailbreak en BFF (`ignore previous`,
  `system:`, `</s>`, etc.) → reject 422 con código `prompt-policy-violation`.
- El BFF nunca propaga al cliente final el system prompt ni metadatos
  internos (`prompt_version`, `model_id`, `correlation_id`) que no estén
  contemplados explícitamente en su contrato OpenAPI.
- Los logs no contienen el contenido completo del prompt; solo hash + tamaño
  + `prompt_version` + `correlation_id`.

## Telemetría obligatoria

Las métricas Prometheus expuestas por `ai-engine` (ver
`docs/architecture/decision-records/0009-llmops-and-ai-evaluation.md`) cubren:

- `ai_engine_llm_errors_total{kind="generation"}` para fallos de validación
  o del modelo.
- `ai_engine_llm_fallback_total` para activaciones de fallback (incluye
  rechazo por prompt policy si se cae a respuesta degradada).
- `ai_engine_llm_tokens_total{direction}` para detectar prompts anómalos
  (picos de tokens entrantes).

Si una alerta dispara fallback sostenido (>5 min con `rate(...) > 0.1`), abrir
incidente y revisar los últimos `prompt_version` desplegados.

## Cambios de prompt: checklist

1. Editar plantilla en `ai_engine/games/prompts.py`.
2. Bumpear `PROMPT_VERSIONS[<game>]` (semver simple `vN`).
3. Añadir/actualizar caso en `src/tests/eval/datasets/baseline.yaml`.
4. Ejecutar `pytest -m eval` localmente y verificar `success_rate == 1.0`.
5. Commit con prefijo `feat(prompts)` o `fix(prompts)` y referenciar el
   `PROMPT_VERSIONS` resultante en el cuerpo del mensaje.

## Referencias

- ADR 0008 — DevSecOps + SSDLC.
- ADR 0009 — LLMOps y eval harness.
- `docs/operations/secrets-rotation-policy.md`.
- OWASP Top 10 for LLM Applications (LLM01: Prompt Injection).
