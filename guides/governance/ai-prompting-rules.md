# AI Prompting Rules

Last updated: 2026-04-23.

Cross-cutting policy against prompt injection and model abuse for `ai-engine`,
the BFFs (`bff-mobile`, `bff-backoffice`) and any service that builds prompts
before invoking the runtime. Complements ADR 0009 (LLMOps) and ADR 0008
(DevSecOps + SSDLC).

## Principles

1. **Explicit trust boundary.** Content coming from the user, from the RAG
   dataset, from external sources or from internal state is treated as
   *untrusted* unless it is a code constant reviewed in commit.
2. **System / user separation.** System instructions live in
   `ai_engine/games/prompts.py` (versioned through `PROMPT_VERSIONS`). User
   input is only inserted inside delimited blocks.
3. **No reflective execution.** The engine never interprets model output as
   operational instruction (no `eval`, no tool calls derived from generated
   text, no shell-out).
4. **Least surprise.** Any new template or substantive change to a template
   must bump `PROMPT_VERSIONS[<game>]` and update the evaluation datasets
   under `src/tests/eval/datasets/`.

## Mandatory rules in `ai-engine`

- Every active template lives in `ai_engine/games/prompts.py` and exposes its
  version through `get_prompt_version(game_type)`.
- User input is wrapped with explicit tags:
  `<<USER_INPUT>>...<</USER_INPUT>>`. The model is instructed to treat that
  content as data, not as orders.
- The system prompt explicitly forbids:
  - Ignoring previous instructions.
  - Revealing the system prompt or environment keys.
  - Producing shell commands, executable code targeting another system or
    external URLs that are not in the allow-list.
- RAG documents are tagged with `[doc:<id>]` and the model must cite the
  source or return `unknown` if it cannot find evidence.
- Outputs are validated against the per-game schema (`quiz`, `word-pass`,
  `true_false`) before being returned to the client. If validation fails,
  `errors_total{kind="generation"}` is recorded and the fallback is applied.

## Mandatory rules in BFFs

- Maximum input size per field (`topic`, `prompt`, etc.) enforced in the BFF
  before invoking the engine (default: 512 characters for topics, 4 KB for
  free-text fields).
- Basic blacklist of jailbreak patterns in the BFF (`ignore previous`,
  `system:`, `</s>`, etc.) → reject with HTTP 422 and code
  `prompt-policy-violation`.
- The BFF never propagates the system prompt or internal metadata
  (`prompt_version`, `model_id`, `correlation_id`) to the final client unless
  it is explicitly part of its OpenAPI contract.
- Logs do not contain the full prompt content; only hash + size +
  `prompt_version` + `correlation_id`.

## Defense-in-depth in `ai-engine`

`ai-engine` runs the heuristic jailbreak classifier defined in
`ai_engine.safety.jailbreak` **before** every `POST /generate` call (and
its SDK variant). When a prompt matches a blocking pattern the request is
rejected with HTTP 422 (`Prompt rejected by safety policy ...`) and the
generator is **never** invoked. The block is recorded with metadata
`safety_block_reason`, `safety_score`, `safety_categories`, exposed via the
counters `ai_engine_prompt_safety_blocks_total` and
`ai_engine_prompt_safety_blocks_by_reason_total{reason="..."}`.

Tunable through environment variables:

- `AI_ENGINE_SAFETY_ENABLED` (default `true`): kill switch for the classifier.
- `AI_ENGINE_SAFETY_BLOCK_THRESHOLD` (default `1.0`): minimum aggregated
  weight required to block.
- `AI_ENGINE_SAFETY_FLAG_THRESHOLD` (default `0.4`): minimum weight to mark
  the prompt as suspicious without blocking it.

Regression dataset lives in
`ai-engine/src/tests/test_safety/test_jailbreak.py` and must be extended
when new attack patterns are observed in production telemetry.

## Mandatory telemetry

The Prometheus metrics exposed by `ai-engine` (see
`docs/architecture/decision-records/0009-llmops-and-ai-evaluation.md`) cover:

- `ai_engine_llm_errors_total{kind="generation"}` for validation or model
  failures.
- `ai_engine_llm_fallback_total` for fallback activations (including rejections
  by the prompt policy that degrade to a fallback response).
- `ai_engine_llm_tokens_total{direction}` to detect anomalous prompts (spikes
  in inbound tokens).

If an alert detects sustained fallback (>5 min with `rate(...) > 0.1`), open
an incident and review the most recently deployed `prompt_version`s.

## Prompt changes: checklist

1. Edit the template in `ai_engine/games/prompts.py`.
2. Bump `PROMPT_VERSIONS[<game>]` (simple `vN` semver).
3. Add or update a case in `src/tests/eval/datasets/baseline.yaml`.
4. Run `pytest -m eval` locally and confirm `success_rate == 1.0`.
5. Commit using prefix `feat(prompts)` or `fix(prompts)` and reference the
   resulting `PROMPT_VERSIONS` entry in the message body.

## References

- ADR 0008 — DevSecOps + SSDLC.
- ADR 0009 — LLMOps and eval harness.
- `docs/operations/secrets-rotation-policy.md`.
- OWASP Top 10 for LLM Applications (LLM01: Prompt Injection).
