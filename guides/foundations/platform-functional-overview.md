# Platform Functional Overview

Last updated: 2026-11-14.

## Purpose

Provide a high-level inventory of the main AxiomNode platform capabilities across all repositories.

This document is intentionally summary-level. It is meant to support later repository-by-repository or capability-by-capability analysis without mixing implementation detail, rollout history, or backlog content into the current-state documentation set.

## Scope

The workspace contains four broad kinds of assets:

- runtime services that serve end users or operators
- platform services that package, observe, secure, or document the runtime
- contract and SDK repositories that keep integrations aligned
- auxiliary repositories and scripts used for branding or data preparation

## Functional map by layer

| Layer | Main repositories | Functional outcome |
|---|---|---|
| Public edge | `api-gateway` | Stable client-facing entry point and public/internal edge routing |
| Experience | `backoffice`, `bff-backoffice`, `bff-mobile`, `mobile-app` | Channel-specific operator and player experiences |
| Domain runtime | `microservice-users`, `microservice-quizz`, `microservice-wordpass` | Identity, gameplay data, and curated game inventory |
| AI capability | `ai-engine` | AI generation, RAG, ingest, observability, and model execution |
| Delivery and governance | `platform-infra`, `observability-platform`, `secrets`, `docs` | Deployment, monitoring, secret governance, and shared documentation |
| Contracts and reuse | `contracts-and-schemas`, `shared-sdk-client` | Canonical contracts and reusable integration primitives |
| Auxiliary assets | `runtime-seed`, `org-porfile` | Dataset preparation and organization profile assets |

## End-user and operator capabilities

### Player-facing capabilities

- mobile gameplay for quiz and word-pass flows
- retrieval of curated random game models
- authenticated player profile access and updates
- offline gameplay continuity with later event synchronization

### Operator-facing capabilities

- operational dashboards for service health and traffic
- runtime inspection of services, metrics, logs, and data inventories
- control of the ai-engine llama target (read, probe, set, reset)
- management of generation workflows and curated content inventories

### Platform-facing capabilities

- automatic build-and-deploy chain for covered runtime services
- shared contract validation and SDK generation
- centralized secrets governance and distribution
- dashboards, alerts, and architecture/operations documentation

## Repository-by-repository summary

### `api-gateway`

**Role:** Public edge boundary for mobile and backoffice traffic.

**Main capabilities:**

- exposes stable public routes for mobile and backoffice clients
- applies edge concerns such as auth, CORS, tracing, and forwarding policy
- routes traffic to `bff-mobile` and `bff-backoffice`
- proxies internal AI generation, ingest, catalogs, health, and stats traffic
- resolves the `ai-engine` upstreams from environment variables only; it holds no runtime override or persisted routing state

**Main dependencies:** `bff-mobile`, `bff-backoffice`, active `ai-engine-api`, active `ai-engine-stats`

### `bff-mobile`

**Role:** Mobile-focused backend-for-frontend.

**Main capabilities:**

- exposes mobile-oriented quiz and word-pass endpoints
- translates public mobile requests into downstream service calls
- keeps mobile clients isolated from domain-service topology changes
- shapes smaller payloads aligned with mobile use cases
- coordinates player profile and gameplay flows for the mobile channel

**Main dependencies:** `microservice-quizz`, `microservice-wordpass`, `microservice-users`

### `mobile-app`

**Role:** Kotlin Multiplatform client application for end users.

**Main capabilities:**

- delivers the player-facing mobile experience
- supports quiz and word-pass gameplay flows
- integrates Firebase-based authenticated identity flows
- uses shared contracts to stay aligned with the public edge API
- supports Android today and shared KMP/iOS-ready code paths

**Main dependencies:** `api-gateway`, Firebase, shared contract definitions

### `backoffice`

**Role:** React operator console for runtime inspection and control.

**Main capabilities:**

- renders dashboards for service status, traffic, and diagnostics
- exposes operational controls for AI targeting and runtime changes
- supports inspection of service inventories, logs, metrics, and health data
- presents operator-oriented runtime summaries and the packaged backoffice version
- gates access by roles such as `SuperAdmin`, `Admin`, `Inspector`, `Viewer`, and `Gamer`

**Main dependencies:** `api-gateway`, `bff-backoffice`

### `bff-backoffice`

**Role:** Operator control plane and admin backend-for-frontend.

**Main capabilities:**

- aggregates and normalizes data from internal services for backoffice screens
- exposes admin APIs for auth, monitoring, service inspection, and generation flows
- proxies the ai-engine llama target control (read, probe, set, reset) to `ai-engine-api`; it keeps no routing state
- provides AI observability surfaces for operators

**Main dependencies:** `microservice-users`, `microservice-quizz`, `microservice-wordpass`, `ai-engine`, `api-gateway`

### `microservice-users`

**Role:** Identity, player profile, leaderboard, and gameplay-event service.

**Main capabilities:**

- manages authenticated player profile reads and updates
- ingests gameplay telemetry and events
- computes or serves leaderboard and monitoring aggregates
- exposes private operational metrics and logs endpoints
- integrates Firebase session validation with development fallback modes

**Main dependencies:** PostgreSQL, Firebase-related auth configuration

### `microservice-quizz`

**Role:** Quiz domain service.

**Main capabilities:**

- orchestrates quiz generation through `ai-engine`
- validates quiz payloads and domain constraints
- persists generated or curated quiz models in PostgreSQL
- serves random or filtered quiz inventory to upstream BFFs
- exposes health and private diagnostics/documentation surfaces

**Main dependencies:** PostgreSQL, `ai-engine-api`, `ai-engine-stats`

### `microservice-wordpass`

**Role:** Word-pass domain service.

**Main capabilities:**

- orchestrates word-pass generation through `ai-engine`
- validates domain payloads such as letters, hints, and answers
- persists generated or curated word-pass models in PostgreSQL
- serves reusable inventory for mobile and operator workflows
- exposes health and private diagnostics/documentation surfaces

**Main dependencies:** PostgreSQL, `ai-engine-api`, `ai-engine-stats`

### `ai-engine`

**Role:** Internal AI capability service.

**Main capabilities:**

- provides AI generation APIs for quiz, word-pass, and related game content
- supports RAG ingestion and retrieval building blocks
- manages model execution through a llama runtime
- exposes a stats and observability API for AI events and aggregates
- supports split runtime topologies where API, stats, and model execution may be separated
- persists effective model-target information so routing can survive restart

**Main dependencies:** Redis, llama runtime, model assets, internal consumers such as quiz and word-pass services

### `contracts-and-schemas`

**Role:** Canonical contract repository.

**Main capabilities:**

- stores versioned OpenAPI specifications
- stores shared JSON Schemas and event contracts
- provides examples and validation inputs for integrations
- acts as the source of truth for cross-repository request/response semantics
- enables SDK generation and CI contract validation

**Main consumers:** gateways, BFFs, domain services, SDK generation pipelines

### `shared-sdk-client`

**Role:** Shared client and integration utility repository.

**Main capabilities:**

- distributes TypeScript SDK artifacts derived from platform contracts
- provides shared proxy, forwarding, and client helpers
- centralizes reusable catalogs and validation helpers
- supports integration consistency across runtime repositories
- publishes reusable packages for downstream services and BFFs

**Main dependencies:** `contracts-and-schemas`

### `platform-infra`

**Role:** Delivery and deployment orchestration repository.

**Main capabilities:**

- owns Kubernetes manifests and overlays per environment
- builds and publishes runtime images to GHCR
- builds and publishes images for covered services on dispatch; deploys are manual-only (`deploy.yaml` `workflow_dispatch`)
- validates deployment inputs and rollout steps
- supports local full-stack or partial-stack development tooling
- provides optional environment overlays for alternative runtime topologies

**Main consumers:** all runtime repositories that publish images

### `observability-platform`

**Role:** Shared observability asset repository.

**Main capabilities:**

- stores dashboards, alerts, and monitoring stack assets
- defines visualization and alerting baselines by service or domain
- supports metrics/log/traces operating model for the platform
- keeps reusable monitoring configuration under version control

**Main consumers:** operators, staging/prod monitoring workflows, backoffice diagnostics

### `secrets`

**Role:** Centralized secrets-governance repository.

**Main capabilities:**

- stores and organizes secret material by repository and environment
- enforces placeholder synchronization and secret hygiene rules
- prepares runtime secret bundles for local or automated workflows
- supports bootstrap, audit, and rotation-related tooling
- reduces drift between tracked configuration and real secret sources

**Main consumers:** all runtime and platform repositories

### `docs`

**Role:** Cross-repository documentation source of truth.

**Main capabilities:**

- documents platform architecture and repository topology
- documents deployment policy, CI/CD behavior, and runtime routing rules
- stores operational runbooks and governance baselines
- keeps architecture decisions and shared engineering guidance centralized

**Main consumers:** operators, contributors, maintainers, reviewers

### `runtime-seed`

**Role:** Dataset preparation and bulk-fix utility workspace.

**Main capabilities:**

- generates seed CSV data for quiz and word-pass inventories
- fetches and normalizes source content from external datasets such as OpenTrivia and FrequencyWords
- prepares import-ready variants with and without explicit IDs
- applies repair scripts to existing word-pass data through backoffice APIs
- supports operational clean-up and history-fix tasks outside the main runtime repos

**Main consumers:** content curation and operational data maintenance workflows

### `org-porfile`

**Role:** Organization profile asset repository.

**Main capabilities:**

- stores organization-level profile content and presentation assets
- separates profile/branding concerns from runtime and platform repositories

**Main consumers:** organization profile publication workflows

## Analytical capability matrix

The capability-oriented analytical matrix has been moved to [platform-capability-matrix.md](./platform-capability-matrix.md) so this document can remain readable as the main summary guide.

Use that matrix when the next step is a deeper capability-by-capability review rather than a repository-level orientation.

The individual capability dossiers are indexed in [../capabilities/README.md](../capabilities/README.md).

## Functional dependency highlights

### Core player flow

1. `mobile-app` calls `api-gateway`.
2. `api-gateway` routes to `bff-mobile`.
3. `bff-mobile` calls `microservice-quizz`, `microservice-wordpass`, and player-profile capabilities.
4. Domain services use curated inventory or call `ai-engine` when generating new content.

### Core operator flow

1. `backoffice` calls `api-gateway`.
2. `api-gateway` routes to `bff-backoffice`.
3. `bff-backoffice` aggregates health, logs, metrics, inventory, and AI diagnostics.
4. Llama target changes are forwarded to `ai-engine-api` and affect effective AI generation routing.

### Delivery flow

1. Contract changes originate in `contracts-and-schemas`.
2. Shared integration code is generated or updated in `shared-sdk-client`.
3. Runtime repositories consume those artifacts.
4. Pushes to covered runtime repos dispatch builds to `platform-infra`.
5. `platform-infra` publishes images to GHCR; deployment is a manual `deploy.yaml` dispatch (no staging host exists today).

## Repositories with persisted runtime influence

Some repositories do more than serve requests; they also hold runtime state that changes effective behavior:

- `ai-engine`: active llama/model target (the only one; `api-gateway` and `bff-backoffice` hold no routing state)

This matters because the effective runtime topology can differ from the static Kubernetes manifests.

## Recommended next step

Use this guide as the index for later individual analysis. The next level of study should document, one capability at a time:

- exposed routes and inputs/outputs
- owned persistence and state transitions
- failure modes and observability signals
- deployment and rollback implications
