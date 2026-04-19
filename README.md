# AxiomNode Documentation

Last updated: 2026-04-19.

Central documentation repository for AxiomNode architecture, operations, and platform governance.

## Main sections

### Architecture

- [Target architecture](./architecture/target-architecture.md)
- [Repository map](./architecture/repository-map.md)
- [Shared code extraction audit](./architecture/common-code-extraction-audit-2026-03-24.md)

### Guides

- [Service communication model](./guides/service-communication.md)
- [Local edge integration](./guides/local-edge-integration.md)
- [Backoffice React development](./guides/backoffice-react-development.md)
- [Mobile app development (Firebase)](./guides/mobile-app-development-firebase.md)
- [Migration guide for existing repositories](./guides/migration-existing-repos.md)

### Operations

- [Environments and secrets](./operations/environments-and-secrets.md)
- [Secrets distribution guide](./operations/secrets-distribution-guide.md)
- [Deployment strategy](./operations/deployment-strategy.md)
- [CI/CD workflow map](./operations/cicd-workflow-map.md)
- [Runtime routing and service targeting](./operations/runtime-routing-and-service-targeting.md)
- [AI services recovery runbook](./operations/ai-services-recovery-runbook.md)

### Next steps

- [Next steps index](./next-steps/README.md)

### Historical reference

- [Implementation phases](./roadmap/implementation-phases.md)

## Recommended reading order

1. Start with architecture and repository map.
2. Read deployment strategy and CI/CD workflow map before changing delivery behavior.
3. Define or update contracts in `contracts-and-schemas`.
4. Regenerate SDK artifacts in `shared-sdk-client`.
5. Implement runtime changes in service repositories.
6. Review runtime routing implications when changes affect AI topology or service targeting.
7. Roll out through `platform-infra` and observe via `observability-platform`.

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

## Design assets

Brand and UI design assets are maintained in `design/`.
