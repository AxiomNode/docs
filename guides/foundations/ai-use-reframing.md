# AI Use Reframing

Last updated: 2026-05-05.

## Purpose

This document reframes how AxiomNode should use AI for the TFM and for the
platform itself.

The previous framing treated generative AI as if it were already a reliable
runtime capability. That is too optimistic. The current implementation has
valuable architecture around `ai-engine`, observability, prompts, routing, and
generation flows, but that does not prove that the produced educational content
is good, stable, safe, or useful enough to be a player-facing dependency.

The new position is: **AI is an assisted content-production and evaluation
capability, not the source of truth of the product.**

## Problem Diagnosis

The current AI story is weak if it is presented as "the platform uses an LLM to
generate games". That is not enough for a TFM and it is not enough for a real
system.

Main problems:

- semantic quality is not proven by the current eval harness because canned
  responses validate structure, not real educational value
- runtime generation is too fragile to be trusted by player-facing flows
- RAG, cache, llama targeting, prompt versions, and metrics are useful platform
  pieces, but they do not automatically make the generated content correct
- fallback behavior can hide failure if it returns plausible but low-value
  content
- operator routing controls make the platform flexible, but they also increase
  the chance that effective AI behavior differs from what the deployment shows
- there is no explicit approval boundary between an AI draft and playable
  inventory

The TFM should not hide these problems. It should use them as the engineering
case study.

## Reframed Thesis

A stronger TFM angle is:

> AxiomNode studies how to integrate unreliable generative AI into an
> educational game platform as a controlled content-production assistant, with
> evaluation, curation, observability, security, and rollback boundaries.

The main research and engineering question becomes:

> How can generative AI support educational content creation without becoming an
> unverified runtime dependency for players or operators?

## Product Rule

The product should serve **approved inventory**, not raw AI output.

That means:

- mobile gameplay should prefer persisted, validated quiz and word-pass content
- AI generation should create candidate content, not final content
- candidate content should pass automated validation before persistence
- high-risk or low-confidence candidates should require operator review
- every persisted AI-generated item should keep provenance metadata
- runtime player flows should degrade to curated inventory or a clear empty
  state, not to improvised generation

This turns AI from a fragile promise into a measurable production pipeline.

## Recommended AI Role Model

| AI role | Keep? | Boundary |
|---|---|---|
| Candidate content generator | Yes | Generates drafts for quiz and word-pass inventory, never directly trusted as final player content. |
| Semantic evaluator | Yes | Scores candidate content using rubric-based checks and optional LLM-as-judge experiments. |
| RAG over platform docs | Optional | Useful as a TFM extension after content quality is under control; not core product value. |
| Developer assistant | Yes | Used for implementation, review, docs, and tests, with explicit human review evidence. |
| Live player-facing generator | No by default | Only acceptable behind feature flags, strict evaluation, clear timeout, and safe fallback. |
| Autonomous agent | No | Too broad for the current project and not justified by the product need. |

## Target Content Pipeline

The preferred AI flow is a content factory:

1. An operator or scheduled job requests candidate generation for a category,
   language, difficulty, and game type.
2. `ai-engine` builds the prompt with versioned templates and optional retrieved
   references.
3. The model returns candidate content.
4. `ai-engine` validates schema, language, item count, answer consistency, and
   basic safety rules.
5. A semantic evaluation pass scores educational usefulness, factuality,
   difficulty fit, diversity, and playability.
6. Low-confidence content is rejected or marked for review.
7. Approved content is persisted by `microservice-quizz` or
   `microservice-wordpass` with provenance fields.
8. Mobile and backoffice consumers read from persisted inventory.
9. Metrics connect each item back to `prompt_version`, model target, eval score,
   latency, fallback state, and reviewer decision when applicable.

## Minimum Provenance Fields

Generated content should become auditable. At minimum, persisted generated rows
should be able to answer:

- which `prompt_version` produced it
- which model or runtime target produced it
- which language, category, and difficulty were requested
- whether RAG context was used
- which automated validators passed or failed
- what semantic score or rubric result was assigned
- whether a human approved, edited, or rejected it
- when it was created and by which generation run

These fields do not all need to be exposed to players. They are operational and
TFM evidence.

## Evaluation Rubric

The first serious AI improvement should be evaluation, not more generation.

Suggested criteria:

| Criterion | Meaning | Example failure |
|---|---|---|
| Schema validity | Output matches the game contract. | Missing correct answer or malformed choices. |
| Language correctness | Output uses the requested language. | Spanish category generates English questions. |
| Educational usefulness | The item teaches or reinforces something meaningful. | Trivial, vague, or nonsensical question. |
| Factuality | Facts are correct for the topic. | Wrong answer marked as correct. |
| Difficulty fit | Content matches the requested difficulty. | Beginner item uses expert-only concepts. |
| Playability | The game can be played without ambiguity. | Two choices are equally correct. |
| Diversity | The batch avoids near-duplicates. | Five questions ask the same fact. |
| Safety | Content avoids unsafe, private, or inappropriate material. | Prompt injection leaks or toxic content. |

## Architecture Implications

This reframing changes the platform priorities:

- `ai-engine` should optimize for measured candidate quality, not just route
  availability
- quiz and word-pass services remain the source of truth for playable inventory
- BFFs and mobile routes should avoid live generation as the default user path
- backoffice should become the natural surface for generation review,
  diagnostics, and approval
- observability should track quality and approval outcomes, not only latency and
  error counters
- contracts should distinguish candidate, approved, rejected, and playable
  content states if those states become API-visible

## TFM Implementation Plan

### P0 - Stop Overclaiming AI

- Document that AI is currently an experimental content-production subsystem.
- Treat existing runtime generation as a capability under evaluation, not as a
  finished feature.
- Make the TFM explicit about current limitations.

### P1 - Build The Evaluation Core

- Add real-model eval cases for quiz and word-pass.
- Produce a report with rubric scores and failure examples.
- Track prompt version, model target, latency, fallback, and validation errors.
- Keep CI non-blocking until the baseline is known.

### P1 - Turn Generation Into Curation

- Add or document candidate states before playable persistence.
- Add provenance metadata for generated rows.
- Require automated validation before content is eligible for random inventory.

### P2 - Add Human Review Where It Matters

- Use backoffice as the review surface for low-confidence or high-impact
  generated content.
- Record approval, rejection, or edit decisions.
- Use reviewed examples as TFM evidence.

### P3 - Add Optional Documentation RAG

- Only after game-content quality is controlled, prototype RAG over platform
  docs as a secondary TFM extension.
- Evaluate it with cited-source accuracy and refusal behavior.

## What Not To Do

- Do not add more AI features before measuring whether current generation is
  useful.
- Do not make live LLM calls part of the default player flow.
- Do not present canned eval success as proof of semantic quality.
- Do not use RAG as decoration if the corpus and answer quality are not
  evaluated.
- Do not hide failures behind fallback content that looks successful.

## Success Criteria

The reframed AI work is successful when the TFM can show:

- at least one real-model evaluation report with failures and improvements
- a clear boundary between candidate AI content and playable inventory
- traceability from generated content to prompt, model, evaluation, and approval
- observable failure modes and fallback behavior
- one end-to-end path where approved AI-assisted content reaches the product
  through persisted inventory, not raw runtime output

## Related Documents

- [AI plugin architecture](./ai-plugin-architecture.md)
- [TFM practice matrix](./tfm-practice-matrix.md)
- [AI content generation](../capabilities/ai/ai-content-generation.md)
- [AI observability and diagnostics](../capabilities/ai/ai-observability-and-diagnostics.md)
- [AI prompting rules](../governance/ai-prompting-rules.md)
- [Critical workflow smoke matrix](../validation/critical-workflow-smoke-matrix.md)