# Platform Capability Matrix

Last updated: 2026-11-14.

## Purpose

Provide a capability-oriented analytical view of AxiomNode that complements the higher-level repository summary in `platform-functional-overview.md`.

Use this document when the goal is to analyze a capability in terms of inputs, outputs, persistence, observability, and risk rather than to understand repository ownership at a glance.

## Analytical capability matrix

| Capability | Main owners | Typical inputs | Main outputs | Owned persistence/state | Main observability signals | Main risks/failure modes |
|---|---|---|---|---|---|---|
| Public edge routing | `api-gateway` | mobile/backoffice HTTP requests, auth headers, correlation IDs | stable public responses, forwarded traffic to BFFs, internal AI proxy calls | none (the gateway keeps no routing state) | gateway health, edge logs, forwarding errors, correlation-id traces | auth rejection, bad upstream target, timeout to BFF, route drift |
| Mobile gameplay orchestration | `bff-mobile`, `mobile-app` | player actions, bearer token, category/language filters | playable quiz and word-pass sessions, profile reads/updates, synced events | mobile local state, BFF request shaping, player continuity on client | mobile app logs, gateway/BFF traces, sync responses | contract mismatch, offline sync divergence, invalid payload shaping |
| Player identity and profile | `microservice-users`, `bff-mobile`, `bff-backoffice` | Firebase tokens, profile updates, gameplay event batches | player profile, stats snapshots, session-linked user identity | PostgreSQL user/profile/event data | auth/session logs, leaderboard reads, stats endpoints | invalid auth, duplicate events, inconsistent profile aggregation |
| Quiz inventory and generation | `microservice-quizz`, `ai-engine` | generation requests, category/language filters, ingest payloads | quiz models, curated/random quiz inventory, generation results | PostgreSQL quiz inventory/history | health, private docs, monitor stats, generation logs | invalid generated payload, persistence failure, empty inventory, AI timeout |
| Word-pass inventory and generation | `microservice-wordpass`, `ai-engine` | generation requests, language/category filters, ingest payloads | word-pass models, curated/random inventory, generation results | PostgreSQL word-pass inventory/history | health, private docs, monitor stats, generation logs | malformed hints/answers, persistence failure, AI timeout, stale data |
| AI content generation | `ai-engine-api`, llama runtime | generation prompts, ingest requests, internal API keys, active model target | validated AI payloads, generation artifacts, ingest results | active llama target, cache state, model/runtime state | `/health`, stats API, AI event collector, cache/runtime metrics | unreachable llama target, invalid JSON output, resource pressure, cache degradation |
| AI observability and diagnostics | `ai-engine-stats`, `bff-backoffice`, `backoffice` | AI event streams, stats queries, operator diagnostics requests | aggregated AI stats, diagnostics views, operator summaries | collector state (no routing state in the BFF) | stats endpoints, dashboards, backoffice diagnostic views | partial stats visibility, stale routing picture, split-topology confusion |
| Runtime routing control | `bff-backoffice`, `ai-engine` | operator llama target changes (set, probe, reset) | effective llama target, updated runtime topology | ai-engine persisted llama target (`AI_ENGINE_LLAMA_TARGET_STATE_FILE`) | `GET /v1/backoffice/ai-engine/target`, probe route, health checks after change | effective topology differs from manifests, wrong host configured, restart-state mismatch |
| Backoffice operational diagnostics | `backoffice`, `bff-backoffice` | operator auth/session, metrics/logs/data queries | dashboards, service metrics/logs, packaged backoffice version | UI state | service metrics/logs endpoints, backoffice dashboards | missing proxy route, partial downstream visibility, role-gating mistakes |
| Contract governance | `contracts-and-schemas`, `shared-sdk-client` | API design changes, schema updates, examples, CI validation inputs | OpenAPI specs, JSON schemas, event contracts, generated SDK artifacts | versioned schema files and generated SDK outputs | CI contract validation, SDK generation jobs | contract drift, SDK lag, incompatible changes between repos |
| Deployment and rollout automation | `platform-infra` | service-repo dispatches, manifests, image tags, environment overlays | GHCR images, manual deploys (`deploy.yaml` `workflow_dispatch`; no staging host today), deployment receipts | manifests, overlays, workflow state, release metadata | GitHub Actions runs, rollout logs, smoke checks | build failure, deploy failure, overlay mismatch, staged image not healthy |
| Monitoring and alerting | `observability-platform` | service metrics, logs, traces, alert definitions | dashboards, alert rules, monitoring stack assets | dashboard JSON/YAML, alert rules, stack configuration | Prometheus/Grafana/alert outputs, dashboard panels | missing telemetry, noisy alerts, blind spots in runtime coverage |
| Secrets governance | `secrets` | environment secret values, placeholder definitions, audit scripts | prepared secret bundles, validated placeholders, distribution material | secret inventories and environment-specific secret sources | audit scripts, placeholder validation outputs, workflow checks | drift between placeholders and real secrets, accidental exposure, broken environment bootstrap |
| Documentation and engineering guidance | `docs` | architecture changes, workflow updates, repository facts | architecture guides, runbooks, governance rules, cross-repo summaries | versioned Markdown docs and ADRs | markdown-link checks, contributor usage | stale guidance, undocumented cross-repo changes, conflicting repo-local docs |
| Seed-data preparation and repair | `runtime-seed` | external source datasets, backoffice API credentials, cleanup parameters | CSV seed files, repaired records, bulk-fix operations | generated CSV artifacts and repair scripts | script output, backoffice API responses, import results | poor source quality, accidental mass update, mismatched import format |
| Organization profile publishing | `org-porfile` | branding/profile content | organization profile assets | repository-stored profile content | repository review and publication results | stale branding content, low operational impact but poor public presentation |

## How to use this matrix

For the later individual study, each capability should be expanded with the same structure:

- functional objective
- owning repositories and boundaries
- entrypoints and contracts
- persistence and runtime state
- observability evidence
- common failure modes
- deployment and rollback implications

Use [capability-analysis-template.md](../templates/capability-analysis-template.md) as the standard starting point for those individual studies.

The detailed dossier set for every capability now lives under [../capabilities/README.md](../capabilities/README.md).
