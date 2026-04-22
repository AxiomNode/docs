# ADR 0006 - Centralized build and deploy via platform-infra

- Status: Accepted
- Date: 2026-04-22

## Context

Each service repo needs to ship images and reach k3s. Letting every repo own its own deploy pipeline duplicated logic, fragmented secrets, and made overlay management inconsistent.

## Decision

Service repos own correctness checks (lint, tests, audit). On `main`, they dispatch to `platform-infra` (`build-push.yaml`), which:

- builds multi-stage images with non-root runtime user,
- publishes to GHCR with `:sha` and `:stg` (and optional `:prod` on controlled dispatch),
- triggers `deploy.yaml` to apply the kustomize overlay for the target environment,
- runs smoke checks.

## Alternatives considered

1. Per-repo deploy workflows: rejected; secret sprawl and overlay drift.
2. Pull-based GitOps tool (Argo/Flux): considered; deferred until current chain shows operational pain points; current solution is simpler.

## Consequences

- Single source of deploy policy.
- Cross-repo dispatch requires PAT (`PLATFORM_INFRA_DISPATCH_TOKEN`); secret rotation policy in `secrets`.
- `platform-infra` becomes a critical dependency for all runtime repos.

## Related

- [Deployment strategy](../../operations/deployment-strategy.md)
- [CI/CD workflow map](../../operations/cicd-workflow-map.md)
