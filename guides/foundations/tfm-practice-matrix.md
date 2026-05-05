# TFM Practice Matrix

Last updated: 2026-05-05.

## Purpose

This document maps the AxiomNode platform against the practices covered by the
Master in AI-assisted Software Development. It is intended as a practical TFM
planning and evidence tool: it shows what the project already demonstrates,
where the strongest learning opportunities remain, and how each improvement can
be validated without turning the platform documentation into a roadmap.

Use this matrix when deciding which technical work best supports the TFM
argument. Do not treat every row as required scope; prefer a small set of
well-measured exercises over many shallow additions.

## TFM Thesis Angle

AxiomNode is strongest when framed as:

> A distributed educational gaming platform that studies how to integrate
> unreliable generative AI as a controlled content-production assistant, with
> evaluation, curation, observability, security, and rollback boundaries.

The central research and engineering question should be:

> How can generative AI support educational content creation without becoming an
> unverified runtime dependency for players or operators?

## Current Evidence Snapshot

| Area | Existing AxiomNode evidence | TFM interpretation |
|---|---|---|
| Architecture | C4 views, repository map, use cases, domain model, ADRs, runtime routing docs | The project has enough architectural substance to justify a platform-level TFM, not only a feature demo. |
| AI capability | `ai-engine`, RAG pipeline, llama runtime integration, prompt versions, generation cache, AI stats | AI exists as platform capability, but it should be treated as experimental until real-model quality, provenance, and curation are proven. |
| LLMOps | ADR 0009, eval harness, Prometheus LLM metrics, Grafana dashboard | The operational frame exists, but current evidence is not enough to trust generated content semantically. |
| DevSecOps | ADR 0008, STRIDE threat model, CodeQL/Trivy workflows, prompting rules, secrets rotation policy | Security practices from the master are already applied; prompt-injection evidence can be strengthened. |
| Quality | Unit/integration tests, coverage gates, Playwright smoke, cross-repository smoke matrix | Testing is mature for a TFM; the remaining gap is deeper end-to-end proof of user value. |
| Delivery | Service CI, central build/push/deploy, immutable tags, k3s overlays | CI/CD and containerization are demonstrable and well documented. |
| UX | Backoffice and mobile clients exist, with some smoke coverage | Usability and accessibility are less evidenced than backend/platform quality. |
| AI-assisted development | Repo-level `AGENTS.md`, prompting policy, documentation governance | The process uses AI, but productivity impact is not yet measured as evidence. |

## AI Reframing Principle

The TFM should not claim that AI already works as a reliable product feature.
The stronger and more honest position is documented in
[AI use reframing](./ai-use-reframing.md): AxiomNode should serve approved
inventory, while AI produces evaluated candidate content.

The execution model is documented in
[AI plugin architecture](./ai-plugin-architecture.md): keep `ai-engine-api` and
`ai-engine-stats` centralized during the next development phase, treat the
external `llama-server` as the first thin plugin surface, and evolve toward a
hybrid plugin only after RAG quality, evaluation, and provenance are measured.

In practice:

- mobile gameplay should not depend on live LLM generation by default
- AI output should be treated as draft content until validation and curation pass
- `ai-engine` should be judged by quality reports, not only route availability
- generated rows should preserve prompt, model, eval, and approval provenance
- failures should be visible instead of hidden behind plausible fallback content

## Practice Matrix

| Master block | Current evidence | Main gap | Practice exercise | Repositories | Validation evidence | TFM value |
|---|---|---|---|---|---|---|
| 2. Software engineering | Use cases, domain model, repository boundaries, issue/PR templates | User stories and acceptance criteria are not grouped by channel | Add a channel story map for mobile and backoffice, linked to existing use cases and critical workflows | `docs` | Updated use-case/story-map document; trace to smoke matrix rows | Shows requirements analysis, decomposition, and verification discipline. |
| 3. Software architecture | C4, ADRs, BFF-per-channel, single edge, runtime control plane | Event consistency is documented but not formalized as a design exercise | Model one async flow with idempotency and possible Outbox adoption, preferably gameplay event sync or batch generation | `docs`, `microservice-users`, `microservice-quizz`, `microservice-wordpass`, `contracts-and-schemas` | ADR or architecture note; event schema check; one executable smoke or integration test | Demonstrates distributed architecture trade-offs instead of only synchronous REST. |
| 4. AI fundamentals | `ai-engine`, RAG pipeline, prompt versions, generation schemas | AI generation is not proven useful or reliable enough for direct product trust | Build a real-LLM evaluation dataset and rubric for quiz and word-pass candidate generation | `ai-engine`, `docs`, optional `observability-platform` | JSON/Markdown eval report with pass rate by criterion, failures, prompt version, model target, and latency | Converts AI from a weak feature claim into a measurable engineering problem. |
| 4. AI architecture | Split AI runtime, llama target store, runtime routing controls | The plugin boundary is still informal and mostly means "external llama endpoint" | Formalize thin-plugin mode and define the future hybrid plugin contract | `ai-engine`, `docs`, `platform-infra`, optional `contracts-and-schemas` | Plugin architecture doc; health/capability probe; routing smoke through selected plugin target | Makes the plugin idea concrete without prematurely moving every AI component out of the VPS. |
| 4. AI responsible | Prompting rules and threat model cover some abuse cases | No dedicated responsible-AI policy | Create a Responsible AI policy covering privacy, prompt logging, sensitive content, human review, transparency, and limits | `docs`, `ai-engine`, BFF repos if behavior changes | Policy doc; tests or log checks proving prompt minimization; reviewed examples | Directly applies ethics, privacy, accountability, and safety from the master. |
| 5. Tools | VS Code/Copilot workflow and repo instructions | Tool impact is not captured as evidence | Keep a lightweight AI-assisted development journal for selected TFM tasks | `docs` or personal TFM notes | Table of task, prompt/tool, human review, saved time, defect found, limitation | Makes AI-assisted development defensible and reflective, not anecdotal. |
| 6. AI development flow | `AGENTS.md`, prompting rules, docs as knowledge base | The current AI work needs a controlled content lifecycle before adding new AI surfaces | Define an AI content-factory workflow: draft, validate, score, review, approve, serve from inventory | `ai-engine`, `microservice-quizz`, `microservice-wordpass`, `backoffice`, `docs` | State model or workflow doc; one generated item traced from prompt to approved inventory | Makes AI operationally useful without pretending it is autonomous or always correct. |
| 7. Testing and quality | Coverage gates, unit/integration tests, Playwright smoke, critical workflow matrix | End-to-end proof is still partial for selected workflows | Promote one critical workflow to full end-to-end evidence from edge to persistence and UI/operator surface | `platform-infra`, `secrets`, `backoffice`, relevant services | Smoke result plus service-local tests; evidence attached to workflow matrix | Shows risk-based testing and honest coverage. |
| 7. Metrics and complexity | Coverage thresholds exist in runtime repos | Quality metrics are scattered and not summarized for TFM | Add a TFM quality evidence summary: coverage, smoke status, key workflow gaps, known limitations | `docs` | One concise evidence table linked to existing validation docs | Helps the TFM reader understand quality without reading every repo. |
| 7. Observability | Prometheus/Grafana baseline and AI LLMOps dashboard | Alert coverage is thinner than dashboards | Add one or two AI-specific alert rules for fallback spikes, prompt-safety blocks, or generation errors | `observability-platform`, `ai-engine` | Alert rule validation; dashboard/runbook link | Demonstrates operating an AI subsystem, not only building it. |
| 7. Usability | Backoffice and mobile apps exist; backoffice has smoke | A11y, microcopy, perceived performance are not systematically reviewed | Run a UX/A11y audit for backoffice diagnostics and mobile gameplay start | `backoffice`, `mobile-app`, `docs` | Checklist, before/after screenshots, Playwright/a11y evidence if feasible | Covers the master usability block, currently underrepresented. |
| 8. Infra and cloud | Docker, k3s, overlays, central deploy workflows, immutable tags | DORA/operational delivery metrics are not measured | Derive lightweight DORA-style metrics from GitHub Actions and deploy events | `platform-infra`, `docs` | Script or manual report for deploy frequency, lead time, recovery notes | Connects DevOps theory with real delivery evidence. |
| 8. LLMOps | ADR 0009, metrics, eval harness, dashboard | Model/prompt quality and runtime target quality are not correlated | Add eval reports that record model target, prompt version, latency, fallback, and pass rate | `ai-engine`, `observability-platform`, `docs` | Eval artifact showing quality + latency + model/prompt version | Makes LLMOps concrete and measurable. |
| 9. Security | STRIDE, prompting rules, CodeQL/Trivy, rotation policy, allowlists | Prompt-injection defenses need stronger regression evidence | Extend jailbreak/exfiltration regression tests and document false-positive/false-negative expectations | `ai-engine`, BFF repos if input policy is enforced there | Safety test suite; metrics for blocked prompts; threat model update | Strongly aligns OWASP, SSDLC, and AI-specific security. |
| 10. AI-powered project | Full stack exists: mobile, backoffice, backend, AI, infra | The AI story is too broad and too fragile if it depends on live generation | Demonstrate AI-assisted content curation end to end: candidate generation -> evaluation -> approval -> inventory -> consumption | `ai-engine`, one game microservice, `backoffice` or `mobile-app`, `docs` | Demo script, eval report, persisted provenance, smoke checks, architecture trace | Shows a realistic AI product pattern instead of a simplistic LLM wrapper. |
| 11. Productivity and professional strategy | Retrospective process, governance docs, templates | Personal learning loop is not explicit | Add a short TFM retrospective after each major practice exercise | Personal notes or `docs` if platform-relevant | Retrospective entry with decision, learning, limitation, next adjustment | Shows reflective professional practice, not only technical output. |

## Recommended TFM Work Packages

### WP1 - AI Content Factory And Measurable Quality

Purpose: stop treating generated content as automatically useful and prove that
AI can produce candidates worth curating.

Scope:

- Build a real-LLM evaluation dataset for quiz and word-pass.
- Define a rubric with at least: schema validity, language correctness,
  educational usefulness, difficulty fit, factuality, diversity, and safety.
- Record `prompt_version`, model target, latency, fallback count, and pass rate.
- Distinguish candidate, rejected, reviewed, and approved content states.
- Keep generated content out of player-facing inventory until validation passes.
- Keep the first CI integration non-blocking until thresholds stabilize.

Best repos:

- `ai-engine`
- `docs`
- optional `observability-platform`

Why this is high value: it turns a failing or simplistic AI feature into the
central engineering case study of the TFM.

### WP1B - Plugin Runtime Architecture

Purpose: make the AI plugin a controlled execution boundary instead of an
informal remote LLM URL.

Scope:

- Keep central `ai-engine-api` and `ai-engine-stats` for the next development
  phase.
- Treat `llama-server` as the first thin plugin surface.
- Add or document plugin probes for health, model id, context size,
  quantization, JSON support, and concurrency.
- Use RAG for changing educational/contextual knowledge; reserve fine-tuning
  for later style/schema adaptation after approved examples exist.
- Define the trigger conditions for moving API/RAG/vector store into a full
  plugin bundle.

Best repos:

- `ai-engine`
- `docs`
- optional `platform-infra`
- optional `contracts-and-schemas` if the plugin contract becomes formal API

Why this is high value: it gives the TFM a clear architectural decision about
what belongs in the VPS and what belongs in the movable AI execution module.

### WP2 - Responsible AI and Prompt Security

Purpose: make the AI subsystem defensible from privacy, abuse, and governance
perspectives.

Scope:

- Add a Responsible AI policy for prompts, logs, datasets, generated content,
  model limitations, human review, and operator responsibilities.
- Extend prompt-injection and exfiltration tests beyond known jailbreak strings.
- Document expected false positives and the escalation path when a safe prompt
  is blocked.

Best repos:

- `docs`
- `ai-engine`
- `bff-mobile` and `bff-backoffice` only if BFF input policy changes

Why this is high value: it applies security, ethics, privacy, and AI-specific
risks in a way that evaluators can inspect directly.

### WP3 - One End-to-End Curated AI Demonstration

Purpose: show the platform as a coherent product flow rather than a set of
independent repositories.

Scope:

- Choose one flow: AI candidate generation -> evaluation -> approval ->
  persistence -> inventory selection -> mobile/backoffice consumption ->
  observability evidence.
- Tighten smoke assertions enough to prove the flow did not silently use a
  degraded shortcut.
- Document the demo path and expected failure modes.

Best repos:

- `platform-infra`
- `secrets`
- `microservice-quizz` or `microservice-wordpass`
- `ai-engine`
- `backoffice` or `mobile-app` depending on the chosen surface

Why this is high value: it makes the TFM easy to demonstrate and connects
architecture, CI/CD, observability, and user value.

### WP4 - AI-Assisted Development Evidence

Purpose: show how AI changed the development process itself.

Scope:

- Keep a structured journal for selected tasks.
- Capture prompt patterns, review steps, generated tests, rejected suggestions,
  defects found, and measurable time/quality impact.
- Include at least one negative case where AI output required correction.

Best repos:

- `docs` if generalized as project evidence
- personal TFM notes if it is reflective rather than platform documentation

Why this is high value: it supports the master theme directly and gives the
TFM a reflective, evidence-based layer.

## Suggested Priority

| Priority | Work package | Reason |
|---|---|---|
| P1 | WP1 - AI Content Factory And Measurable Quality | Highest academic and technical leverage; directly addresses the current AI weakness. |
| P1 | WP1B - Plugin Runtime Architecture | Clarifies whether API/stats stay central and turns the plugin into a real architecture. |
| P1 | WP2 - Responsible AI and Prompt Security | Covers the biggest AI-specific risk area. |
| P2 | WP3 - One End-to-End Curated AI Demonstration | Makes the project demonstrable without depending on raw live generation. |
| P2 | UX/A11y audit | Covers an underrepresented master block with visible outcomes. |
| P3 | Documentation RAG assistant | Strong optional extension, but avoid it until core evaluation is solid. |
| P3 | DORA metrics | Useful DevOps evidence, but less central than AI quality and safety. |

## Evidence Checklist For The TFM Report

- Problem statement and users: mobile player, backoffice operator, platform engineer.
- Architecture evidence: C4, repository map, ADRs, runtime topology distinction.
- AI evidence: content-factory flow, RAG/cache behavior, prompt versions, eval results, provenance, approval boundary.
- Quality evidence: tests, smoke matrix, coverage gates, known partial gaps.
- Security evidence: threat model, prompt-injection policy, SAST/SCA, secrets rotation.
- LLMOps evidence: metrics, dashboard, fallback behavior, eval reports.
- Product evidence: one complete flow from generation to consumption.
- Reflection evidence: where AI helped development, where human review corrected it,
  and what limitations remained.

## References

- [Platform architecture](../../architecture/target-architecture.md)
- [Master alignment](../../architecture/master-alignment.md)
- [Quality attributes](../../architecture/quality-attributes.md)
- [Critical workflow smoke matrix](../validation/critical-workflow-smoke-matrix.md)
- [AI prompting rules](../governance/ai-prompting-rules.md)
- [Threat model](../../operations/threat-model.md)
- [Runtime routing and service targeting](../../operations/runtime-routing-and-service-targeting.md)
- [AI use reframing](./ai-use-reframing.md)
- [AI plugin architecture](./ai-plugin-architecture.md)