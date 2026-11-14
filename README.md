# AxiomNode Documentation

Last updated: 2026-05-08.

Central documentation repository for AxiomNode architecture, operations, and platform governance.

## Main sections

### Architecture

- [Architecture index](./architecture/README.md)
- [Platform architecture](./architecture/target-architecture.md)
- [Repository map](./architecture/repository-map.md)
- [Use cases](./architecture/use-cases.md)
- [C4 views (context, containers, components)](./architecture/c4-views.md)
- [Domain model](./architecture/domain-model.md)
- [Key sequence flows](./architecture/key-sequence-flows.md)
- [Quality attribute scenarios](./architecture/quality-attributes.md)
- [Glossary](./architecture/glossary.md)
- [Master alignment (mapping with AI development master)](./architecture/master-alignment.md)
- [Architecture Decision Records](./architecture/decision-records/README.md)

### Guides

- [Guides index](./guides/README.md)
- [Platform functional overview](./guides/foundations/platform-functional-overview.md)
- [Platform capability matrix](./guides/foundations/platform-capability-matrix.md)
- [Documentation architecture](./guides/governance/documentation-architecture.md)
- [Capability analysis template](./guides/templates/capability-analysis-template.md)
- [Architecture document template](./guides/templates/architecture-document-template.md)
- [Operations runbook template](./guides/templates/operations-runbook-template.md)
- [Repository documentation template](./guides/templates/repository-documentation-template.md)
- [Capability dossiers index](./guides/capabilities/README.md)
- [Service communication model](./guides/communication/service-communication.md)
- [Local edge integration](./guides/communication/local-edge-integration.md)
- [Backoffice React development](./guides/experience/backoffice-react-development.md)
- [Mobile app development (Firebase)](./guides/experience/mobile-app-development-firebase.md)
- [Primary validation workflows](./guides/validation/primary-workflows-validation.md)
- [Migration guide for existing repositories](./guides/governance/migration-existing-repos.md)
- [Repository standards adoption guide](./guides/governance/repository-standards-adoption.md)

### Operations

- [Operations index](./operations/README.md)
- [Environments and secrets](./operations/environments-and-secrets.md)
- [Secrets distribution guide](./operations/secrets-distribution-guide.md)
- [Deployment strategy](./operations/deployment-strategy.md)
- [Branch protection policy](./operations/branch-protection-policy.md)
- [GitHub governance baseline](./operations/github-governance-baseline.md)
- [CI/CD workflow map](./operations/cicd-workflow-map.md)
- [Repository minimum standards](./operations/repository-minimum-standards.md)
- [Functionality review baseline](./operations/functionality-review-baseline.md)
- [Test coverage policy](./operations/test-coverage-policy.md)
- [Coverage baseline report](./operations/test-coverage-baseline.md)
- [Runtime routing and service targeting](./operations/runtime-routing-and-service-targeting.md)
- [AI services recovery runbook](./operations/ai-services-recovery-runbook.md)

## Recommended reading order

1. Start with architecture and repository map.
2. Read deployment strategy and CI/CD workflow map before changing delivery behavior.
3. Define or update contracts in `contracts-and-schemas`.
4. Regenerate SDK artifacts in `shared-sdk-client`.
5. Implement runtime changes in service repositories.
6. Review runtime routing implications when changes affect AI topology or service targeting.
7. Roll out through `platform-infra` and observe via `observability-platform`.

## Documentation ownership model

The workspace intentionally separates project-wide documentation from repository-local documentation.

### Documentation kept in this repository

Use this repository for cross-repository concerns:

- current platform architecture and repository topology
- deployment policy and CI/CD chain behavior
- runtime routing behavior that spans multiple repositories
- environment model, secret governance, and operational runbooks
- shared delivery rules, maintenance policy, and architectural constraints

This repository intentionally excludes roadmap, backlog, audit-history, and change-history documents. It should describe the current platform only.

### Documentation kept in each repository

Each runtime or platform repository should document its own concrete implementation surface:

- repository responsibility and runtime role
- local development workflow
- public or private endpoints exposed by that repository
- persistent state owned by that repository
- repository-specific CI gates, smoke checks, and release constraints
- local architecture notes that do not need to be duplicated globally

### Current platform snapshot

| Layer | Repositories | Notes |
|---|---|---|
| Public edge | `api-gateway` | Single public ingress, stable client-facing boundary |
| Experience | `bff-mobile`, `bff-backoffice`, `backoffice`, `mobile-app` | Channel-specific contracts and operator experience |
| Domain runtime | `microservice-users`, `microservice-quizz`, `microservice-wordpass` | Private business services with owned persistence |
| AI capability | `ai-engine` | Generation API, stats API, cache, llama runtime |
| Platform governance | `platform-infra`, `contracts-and-schemas`, `shared-sdk-client`, `secrets`, `observability-platform` | Delivery, contracts, SDKs, secrets, and operations tooling |

Backoffice access is role-gated through `microservice-users`. The current role model includes `SuperAdmin`, `Admin`, `Inspector`, `Viewer`, and `Gamer`; `Inspector` is a read-only review role that can inspect Backoffice role assignments without mutating user permissions.

### Current runtime reality to document explicitly

- The deployed Kubernetes topology and the effective runtime topology are not always identical.
- Only `ai-engine-api` holds persisted runtime targeting state (the llama target); `api-gateway` and `bff-backoffice` hold none.
- Staging may run `ai-engine-api` and `ai-engine-stats` in-cluster while the llama runtime remains external, or it may render the optional full in-cluster AI overlay for controlled diagnostics.
- Documentation must distinguish environment defaults, persisted overrides, and live effective targets.

## Documentation maintenance rules

- Update docs in the same PR/commit window as architecture or workflow changes.
- Avoid undocumented cross-repo decisions.
- Keep contract and SDK documentation synchronized.
- Preserve valid relative links across the repository; CI now checks Markdown link targets.

## Documentation quality expectations

Core architectural and operational documents should be:

- current-state accurate rather than aspirational
- explicit about ownership, scope, and failure boundaries
- linked to the repositories and workflows they describe
- written so that operators and engineers can distinguish static deployment state from effective runtime state
- updated whenever delivery policy, topology, or runtime control behavior changes

Repository-local documentation should additionally be:

- concrete about ports, routes, stored state, and dependencies
- clear about what is owned locally versus inherited from the wider platform
- aligned with the central documents in this repository rather than restating conflicting summaries

## Design assets

Brand and UI design assets are maintained in `design/`.
