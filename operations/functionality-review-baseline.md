# Functionality Review Baseline

Last updated: 2026-05-03.

## Purpose

Provide a conscious, capability-oriented review baseline for AxiomNode focused on three questions:

1. does the functionality work for its intended consumer,
2. is the path reasonably efficient and easy to operate,
3. is there enough evidence to verify that it still works after change.

This document is not a roadmap. It is a current-state review snapshot used to decide where to measure, simplify, harden, or remove functionality.

## Review dimensions

### Efficacy

How confidently the platform can achieve the intended user or operator outcome today.

### Efficiency

How much hop count, duplicated state, manual intervention, or operational complexity the capability currently needs.

### Corroborability

How easy it is to prove that the capability works through tests, smoke checks, route probes, diagnostics, or deployment evidence.

## Rating scale

- `High`: good current state, low ambiguity.
- `Medium`: works, but with visible complexity or evidence gaps.
- `Low`: partial implementation, unclear ownership, or weak verification.

## Capability review matrix

| Capability | Main repositories | Primary access path | Current evidence | Efficacy | Efficiency | Corroborability | Current gaps |
|---|---|---|---|---|---|---|---|
| Public edge routing | `api-gateway`, `bff-mobile`, `bff-backoffice` | mobile/backoffice -> `api-gateway` -> BFF | `api-gateway/src/tests/proxy.test.ts`, `health.test.ts`, `serverRateLimit.test.ts`, workflow `WF-02`, `WF-03`, `WF-06` | High | Medium | Medium | No explicit smoke matrix proving every new edge route through gateway before rollout; persisted AI target can drift from static manifests. |
| Mobile gameplay orchestration | `mobile-app`, `api-gateway`, `bff-mobile`, `microservice-quizz`, `microservice-wordpass`, `microservice-users` | mobile client -> gateway -> `bff-mobile` -> domain services | `GamesHttpClientRoutesTest.kt`, `GamesRepositoryOfflineFallbackTest.kt`, `bff-mobile/src/tests/mobile.test.ts`, workflows `WF-01` to `WF-04` | Medium | Medium | Medium | Android path is strongest, but iOS and JVM auth remain incomplete; no one explicit end-to-end smoke path from client through gateway and BFF into downstream services. |
| Player identity and profile | `microservice-users`, `bff-mobile`, `bff-backoffice`, `mobile-app` | mobile/backoffice -> BFF -> users service | `users.integration.test.ts`, `firebaseAuthService.test.ts`, `userService.test.ts`, workflows `WF-01`, `WF-04` | Medium | Medium | Medium | Auth fallback rules vary by environment; `bff-mobile` keeps a local player snapshot store while `microservice-users` is the system of record, which creates ownership ambiguity. |
| Quiz inventory and generation | `microservice-quizz`, `ai-engine` | BFF/backoffice -> quiz service -> `ai-engine` | `generationService.test.ts`, `gamesRoutes.test.ts`, `aiEngineClient.test.ts`, workflow `WF-05` | High | Medium | Medium | Coverage verification is currently blocked by a failing expectation in the coverage suite; old persisted rows may need explicit revalidation after validator changes. |
| Word-pass inventory and generation | `microservice-wordpass`, `ai-engine`, `runtime-seed` | BFF/backoffice/tooling -> word-pass service -> `ai-engine` or repair path | `generationService.test.ts`, `gamesRoutes.test.ts`, `modelGenerationJob.test.ts`, workflow `WF-05` | High | Medium | Low | Coverage verification is currently blocked; repair and post-generation maintenance are important but still weakly formalized for dry-run and audit evidence. |
| AI content generation | `ai-engine`, `microservice-quizz`, `microservice-wordpass`, `api-gateway` | internal service or operator path -> `ai-engine-api` -> llama runtime | `ai-engine/src/tests/test_api/test_app.py`, `test_games/test_generator.py`, `test_api/test_llama_target_store.py`, large pytest tree | High | Medium | Medium | Strong unit coverage exists, but split-runtime state makes operational proof harder; true-false generation still lacks a clearly documented consumer path. |
| AI observability and diagnostics | `ai-engine-stats`, `bff-backoffice`, `backoffice` | operator -> backoffice -> `bff-backoffice` -> ai stats and service probes | `backoffice` diagnostics panel tests, `bff-backoffice/src/tests/backoffice.test.ts`, `ai-engine` observability tests | Medium | Medium | Medium | Partial-data and degraded-data labeling rules are still not explicit enough; metric ownership across raw telemetry and UI panels remains diffuse. |
| Runtime routing control | `bff-backoffice`, `api-gateway`, `ai-engine` | operator -> backoffice -> `bff-backoffice` -> gateway / ai-engine | `test_llama_target_store.py` and the BFF target tests, workflow `WF-06` | Medium | Low | Medium | The llama target state in `ai-engine-api` is the only runtime state holder affecting topology; audit-trail consistency and concurrent-edit semantics are not yet clear enough. |
| Backoffice operational diagnostics | `backoffice`, `bff-backoffice`, `api-gateway` | operator UI -> gateway -> `bff-backoffice` -> services/k8s/AI | `service-overview-panel.integration.test.tsx`, `service-console-panel.integration.test.tsx`, `bff-backoffice/src/tests/backoffice.test.ts` | Medium | Medium | Medium | The UI is well covered locally, but there is still no single explicit smoke matrix proving the whole diagnostics chain through gateway, BFF, and downstream services. |
| Contract governance | `contracts-and-schemas`, `shared-sdk-client` | schema change -> validation -> SDK generation -> consumer repos | `scripts/validate_contracts.py`, TypeScript generation builds, CI validations | Medium | Medium | Low | `shared-sdk-client/typescript` still has build-only verification and no test or coverage script; some effective contracts remain implicitly service-owned instead of centrally versioned. |
| Deployment and rollout automation | `platform-infra`, covered runtime repos | service repo CI -> `platform-infra` build/push -> deploy | service CI workflows, `platform-infra` validation, smoke scripts such as `smoke-stg-edge.sh` | High | Medium | Medium | Covered services deploy well, but post-deploy smoke depth is uneven by capability and split-AI topologies complicate equivalent validation. |
| Monitoring and alerting | `observability-platform`, runtime repos, `backoffice` | services export telemetry -> dashboards / alerts / operator views | alert validation, service metrics tests in Node services, observability asset repo | Medium | Medium | Low | Critical-path alert coverage is not yet mapped capability by capability; blind spots are still possible even with dashboards and rules present. |
| Secrets governance | `secrets`, all runtime/platform repos | secret source -> prepare-runtime-secrets -> repo-local env/assets | placeholder validation, central workflows, nightly edge smoke | Medium | Medium | Medium | Centralization is strong, but evidence for ownership boundaries and rotation completeness still varies across repositories and environments. |
| Documentation and engineering guidance | `docs`, all repos | maintainer/operator review path | markdown link validation, normalized repo docs, capability dossiers | High | Medium | Medium | The current docs are much more coherent than before, but a reusable functionality-review baseline was missing until this document. |
| Seed-data preparation and repair | `runtime-seed`, `microservice-wordpass`, `microservice-quizz`, `backoffice` | scripts -> service/backoffice APIs or import flows | script outputs, operational docs, CSV artifacts | Medium | Low | Low | The scripts are useful but destructive by nature; dry-run, audit trail, and rollback evidence are still not formalized strongly enough. |
| Organization profile publishing | `org-porfile` | repository content -> publication workflow | repository content review only | Medium | High | Low | Low operational criticality, but repository naming and publication workflow are still weakly standardized. |

## Cross-capability relationship findings

### Capabilities with strong consumer paths

- public edge routing is clearly consumed by `mobile-app` and `backoffice`
- quiz and word-pass inventory capabilities are clearly consumed by `bff-mobile`, `bff-backoffice`, and the operator workflows
- player identity is clearly consumed by both the mobile and operator channels

### Capabilities with partial or asymmetric access

- mobile authentication is asymmetric by platform: Android is real, iOS and JVM are still dev-mode fallback paths
- AI content generation is strongly used by domain services, but some route surface such as true-false generation still lacks a clearly documented consumer lifecycle
- seed-data preparation and repair is operationally important but still depends more on scripts and operator discipline than on productized access paths

### Capabilities with duplicated or layered state

- runtime routing control spans `bff-backoffice`, `api-gateway`, and `ai-engine`, so effective topology can diverge from static manifests
- player-facing continuity spans mobile local persistence, BFF-side player snapshot storage, and `microservice-users` authoritative persistence
- diagnostics surfaces combine UI state, BFF state, runtime routing state, and live downstream health, which makes stale or partial views more likely without explicit labeling

## Most important current gaps

### P0 usability and correctness gaps

- the smoke matrix now exists, but several critical workflows still rely on manual evidence or partial automation across gateway, BFF, and downstream services
- mobile auth support is not equally usable across Android, iOS, and JVM targets
- routing state is split across multiple repositories and needs stronger auditability and recovery semantics

### P1 corroborability gaps

- `microservice-quizz` and `microservice-wordpass` have real tests, but their coverage verification is currently blocked
- `shared-sdk-client/typescript` is used in many runtime paths without a real test or coverage harness
- monitoring and alerting coverage is not yet mapped capability by capability, so some blind spots may stay invisible until incident time

### P1 productization gaps

- seed and repair tooling is powerful but still feels like an operator script surface rather than a verified, auditable maintenance flow
- diagnostics and AI observability still need stronger rules for partial-data labeling and source attribution

## How to use this baseline

Use this document when reviewing whether a capability should be:

- hardened with more tests or smoke checks,
- simplified because it duplicates state or hops,
- exposed better because it exists but is hard to reach,
- removed because the platform no longer has a meaningful consumer path for it.

Track prioritized follow-up actions outside this current-state documentation repository, for example in issues or the active delivery board.