# AI Plugin Architecture

Last updated: 2026-05-05.

## Purpose

This document proposes the next architecture for AxiomNode's AI plugin model.

The original goal was to provide precise project/domain context to an AI system
so it could generate educational game content. The first deployment idea was to
host the full AI runtime on the same VPS model as the rest of the services. The
current direction is better framed as a **plugin execution model**: the platform
keeps product, governance, inventory, and observability responsibilities, while
the model execution capability can live on a local workstation, GPU machine,
alternate VPS, or future packaged plugin runtime.

## Recommendation

Use a **hybrid plugin architecture**:

- keep the public platform, domain services, backoffice control plane, and
  approved inventory in the VPS-hosted platform
- keep `api-gateway`, BFFs, game services, and persistence as the product
  source of truth
- treat the plugin as an AI execution and context-processing unit that can be
  moved, replaced, or disabled
- keep `ai-engine-api` and `ai-engine-stats` in the VPS during the next
  development phase, while the plugin initially exposes the LLM runtime
- evolve toward a fuller plugin bundle only when local context indexes,
  offline mode, GPU locality, or third-party installation require it

In short: **do not move everything into the plugin yet.** First make the plugin
boundary clean and measurable.

## Current Topology

The current architecture already supports split AI:

```text
domain services / gateway
  -> ai-engine-api on VPS or controlled runtime
    -> external llama-server target
  -> ai-engine-stats on VPS or controlled runtime
```

This is useful because `ai-engine-api` remains the policy and orchestration
layer, while the expensive model process can run wherever it is practical.

Current strengths:

- simple internal contract for callers
- centralized auth, prompt policy, validation, cache, and generation telemetry
- backoffice/runtime targeting can repoint the model target
- no need to duplicate API/stats secrets and deployment behavior per plugin

Current weaknesses:

- the plugin boundary is informal: today it is mostly "some llama endpoint"
- the central API sends context over the network to the plugin model runtime
- vector data and model execution are not packaged as one reusable plugin unit
- quality evaluation and provenance are not strong enough yet

## Topology Options

### Option A - Thin Plugin: LLM Server Only

```text
VPS platform
  -> ai-engine-api
  -> ai-engine-stats
  -> plugin llama-server
```

Use this now.

Benefits:

- smallest change from the current architecture
- centralized governance and metrics
- easier contracts and secrets
- plugin can be any OpenAI/llama-compatible server
- best for development while quality is still being solved

Costs:

- retrieved context travels from API to plugin
- the plugin cannot own its local vector index
- less useful for third-party packaging or offline operation

Best fit:

- current development phase
- local GPU/CPU workstation acting as model executor
- experiments with several generator models

### Option B - Full Plugin: API, Stats, RAG, LLM Together

```text
VPS platform
  -> gateway/BFF target override
    -> plugin ai-engine-api
    -> plugin ai-engine-stats
    -> plugin vector store
    -> plugin llama-server
```

Use later only if plugin distribution becomes a real product requirement.

Benefits:

- context index and model execution stay together
- lower latency between retrieval and generation
- easier offline or customer-local execution
- plugin can be versioned as a self-contained AI appliance

Costs:

- duplicates API/stats deployment and secret handling
- harder to aggregate telemetry centrally
- harder to guarantee consistent prompt policy and validation
- requires plugin registration, health, version, and migration rules

Best fit:

- a packaged local AI module for users/operators
- privacy-sensitive corpora that must not leave the plugin host
- GPU-hosted environments where model and retrieval should be co-located

### Option C - Hybrid Plugin Runtime

```text
VPS platform control plane
  -> central product services and inventory
  -> central stats aggregation
  -> plugin runtime bundle
       - generation API adapter
       - local vector store
       - embedding model
       - reranker
       - LLM server
       - local stats buffer
```

This is the target architecture after Option A is stable.

Benefits:

- keeps product truth in the platform
- lets the AI execution unit move independently
- supports local vector indexes when needed
- allows central observability through plugin heartbeats and event forwarding
- avoids making player flows depend directly on live LLM output

Costs:

- requires a formal plugin contract
- requires plugin lifecycle management
- requires a clear distinction between local plugin stats and central stats

Best fit:

- AxiomNode as a platform with optional AI execution modules
- TFM demonstration of runtime topology, LLMOps, and plugin governance

## Decision For The Current Project State

Because the project is still in development and data loss is not a concern, the
right order is:

1. Keep `ai-engine-api` and `ai-engine-stats` in the VPS or central runtime for
   now.
2. Treat the current external `llama-server` as the first plugin surface.
3. Formalize the plugin contract before moving API/stats into it.
4. Build the RAG and evaluation pipeline around candidate content quality.
5. Move API/RAG/vector store into the plugin only after the thin plugin is
   stable and measured.

This avoids premature distribution complexity while preserving the option to
package a full plugin later.

## Plugin Contract

The thin plugin should expose at minimum:

| Endpoint | Purpose |
|---|---|
| `GET /health` | plugin liveness and dependency status |
| `GET /v1/models` | available model list and active model metadata |
| `POST /v1/completions` or `POST /v1/chat/completions` | model generation interface |
| `GET /capabilities` | optional endpoint describing context length, JSON mode, GPU/CPU, quantization, max parallel requests |

`ai-engine-api` now exposes the central read side for this first thin-plugin
contract through `GET /internal/plugin/capabilities`. That endpoint does not
move execution into the plugin; it reports the effective llama target, probes
the OpenAI-compatible model list when available, and returns the static
generation settings known by the central API.

The future full plugin should additionally expose:

| Endpoint | Purpose |
|---|---|
| `POST /ingest` | local corpus ingestion |
| `POST /retrieve` | retrieve top context chunks for a query |
| `POST /generate-candidate` | generate candidate game content with local RAG |
| `POST /evaluate-candidate` | score generated content against a rubric |
| `GET /stats/export` | export buffered plugin telemetry to central stats |

## Recommended Generation Architecture

The strongest architecture is RAG plus validation and curation, not fine-tuning
first.

```text
curated sources
  -> chunking with metadata
  -> embedding model
  -> vector store
  -> retriever + optional reranker
  -> prompt builder with citations/context ids
  -> generator LLM
  -> schema validation
  -> semantic evaluation
  -> candidate approval
  -> persisted playable inventory
```

## RAG Versus Fine-Tuning

### Prefer RAG Now

Use RAG for concrete, changing, project-specific, and educational context.

Reasons:

- the knowledge base can change without retraining a model
- provenance and citations are easier to inspect
- the system can explain which source produced each generated item
- it is cheaper and faster during development
- it aligns with the plugin goal: context can be shipped or rebuilt per plugin

Good corpus sources:

- curated category definitions
- approved quiz and word-pass examples
- educational notes from the target domain
- rejected examples with reasons, if used carefully for evaluation
- platform docs only for optional architecture Q&A, not game generation

### Do Not Use Fine-Tuning As Knowledge Storage

Fine-tuning should not be used to inject factual content or frequently changing
course/domain knowledge.

Use fine-tuning later only for:

- output style stability
- schema discipline
- wording consistency
- adapting a small model to the exact quiz/word-pass format
- reducing prompt length after a strong accepted-content dataset exists

Fine-tuning becomes realistic only after AxiomNode has a dataset of approved and
rejected generations with reviewer/evaluation labels.

## Model Strategy

Use separate models for separate tasks when possible:

| Task | Recommended model type | Notes |
|---|---|---|
| Embeddings | multilingual embedding model | Start with current MiniLM; consider `bge-m3` or `multilingual-e5` if retrieval quality is weak. |
| Reranking | cross-encoder or small reranker | Optional, useful when top-k retrieval is noisy. |
| Generation | instruction-tuned 7B-class model or stronger | Prefer JSON reliability and Spanish/English quality over raw benchmark score. |
| Evaluation | separate judge model or deterministic rubric | Keep it separate from the generator where feasible. |

The generator should be selected by measured output quality, not only by whether
it runs locally.

## Vector Store Strategy

The current in-memory vector store is fine for tests and demos, but the plugin
architecture should move toward a persistent vector store.

Recommended path:

1. Development: in-memory store for tests and fast local feedback.
2. Local plugin: Chroma or Qdrant with a mounted volume.
3. Shared platform: Qdrant if central multi-service retrieval becomes useful.

For a plugin, local Chroma is a practical first step because it keeps the bundle
small. Qdrant is better if the vector database becomes a real shared service.

## Context Design

Context should be specific, compact, and inspectable.

Each chunk should include metadata:

- `game_type`
- `language`
- `category_id`
- `category_name`
- `difficulty_band`
- `source_type`
- `source_id`
- `quality_status`
- `approved_at` when applicable

Generation prompts should receive:

- the requested game type, language, category, and difficulty
- only top-ranked context chunks
- source ids that can be recorded in provenance
- explicit instruction to generate fewer items if context is insufficient

## Quality Gate

Generated content should not become playable until it passes:

1. JSON/schema validation
2. language validation
3. answer consistency validation
4. duplicate detection
5. context-grounding check where possible
6. semantic rubric score
7. optional human approval for low-confidence results

## Stats Placement

For now, keep `ai-engine-stats` centralized.

Reasoning:

- operators need one place to observe all AI activity
- plugin-local stats are useful but should forward to central stats
- central stats survive plugin replacement
- TFM evidence is easier to collect from one stats surface

Future plugin mode can add a local stats buffer, but it should export or forward
events to the central stats API.

## When To Move `ai-engine-api` Into The Plugin

Move `ai-engine-api` into the plugin only when at least one of these is true:

- the plugin must own a private local vector store
- context cannot leave the plugin host
- the plugin must work offline from the VPS
- latency between central API and plugin model becomes a measured bottleneck
- third-party installation requires a self-contained AI module

Until then, keeping the API central is simpler and safer.

## Implementation Roadmap

### Step 1 - Formalize Thin Plugin Mode

- Document external model runtime as plugin mode.
- Add capability/probe checks for model id, context size, JSON mode request, and
  max concurrency. Initial implementation lives in
  `GET /internal/plugin/capabilities` on `ai-engine-api`.
- Keep central API/stats.

### Step 2 - Improve RAG Quality

- Replace or complement in-memory vector storage with persistent Chroma/Qdrant
  for plugin experiments.
- Add metadata-rich chunks.
- Add retrieval evaluation cases.

### Step 3 - Add Candidate Evaluation

- Score generated content with deterministic validators plus rubric checks.
- Record prompt version, model id, sources, retrieval scores, and eval result.

### Step 4 - Add Curation States

- Introduce candidate, rejected, approved, and playable states in the content
  lifecycle.
- Keep mobile routes on approved inventory.

### Step 5 - Package Full Plugin Only If Needed

- Bundle API, stats buffer, vector store, embedding model, reranker, and LLM.
- Add plugin registration and heartbeat toward the central platform.

## TFM Position

This architecture is stronger for the TFM because it does not claim that a local
LLM is magically reliable. It studies the real engineering problem:

> how to provide concrete context to an LLM, measure whether the generated
> content is useful, and integrate that output safely into a distributed product.

## Related Documents

- [AI use reframing](./ai-use-reframing.md)
- [TFM practice matrix](./tfm-practice-matrix.md)
- [Runtime routing and service targeting](../../operations/runtime-routing-and-service-targeting.md)
- [AI content generation](../capabilities/ai/ai-content-generation.md)
- [AI observability and diagnostics](../capabilities/ai/ai-observability-and-diagnostics.md)