# Glossary

Last updated: 2026-04-22.

Common terms used across AxiomNode documentation.

| Term | Meaning |
|---|---|
| Edge | The single public ingress, implemented by `api-gateway`. |
| BFF | Backend-for-Frontend; channel-shaped API tier (`bff-mobile`, `bff-backoffice`). |
| Domain service | Microservice owning a single bounded context with its own database (`microservice-users/quizz/wordpass`). |
| AI subsystem | `ai-engine-api`, `ai-engine-stats`, embedding store, cache, and llama runtime. |
| Llama runtime | LLM model server; may run in-cluster or on an external workstation. |
| Split-AI mode | Runtime configuration where `ai-engine-*` services run in-cluster but llama runs externally. |
| Full in-cluster AI | Optional overlay that runs llama inside k3s for diagnostics. |
| Effective topology | The routing actually in use, including persisted runtime overrides. |
| Declared topology | The topology described by Kubernetes manifests in `platform-infra`. |
| Routing target | Persisted runtime URL for the ai-engine llama upstream (the only runtime routing target). |
| Control plane (runtime) | Backoffice-driven llama target mutation (set, reset) that survives pod restart. |
| Cache key | Composite key `(category_id, embedding_model, corpus_signature, request_shape)` used by `ai-engine-api`. |
| Corpus signature | Stable hash of the curated corpus used for retrieval; participates in cache keys. |
| GHCR | GitHub Container Registry, where service images are published. |
| `:sha` tag | Immutable image tag derived from the build commit. |
| `:stg` / `:prod` | Floating environment tags pinned per overlay; production publication is gated by manual dispatch. |
| Sealed-secret | Encrypted secret object committed to git and decrypted in-cluster. |
| Validation chain | Per-repo CI gates (lint, test, audit, contract validation) that must pass before dispatch. |
| Dispatch | Cross-repo `repository_dispatch` from a service to `platform-infra` to trigger build/push. |
| SDK | `shared-sdk-client`: an internal TypeScript library (shared Zod schemas, HTTP proxy helpers, game-service runtime). Not generated from contracts. |
| Contract | OpenAPI/JSON Schema/event schema in `contracts-and-schemas`. Hand-written and validated by CI; the event schemas are not implemented. |
| Bounded context | DDD term; one microservice = one context. |
| Aggregate | DDD term; consistency boundary inside a bounded context. |
| ADR | Architecture Decision Record (see `decision-records/`). |
