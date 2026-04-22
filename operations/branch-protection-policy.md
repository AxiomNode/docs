# Branch Protection Policy

Last updated: 2026-04-22.

## Intent

This document defines the minimum branch protection expectations for AxiomNode repositories.

It is a governance baseline for the active delivery branch, which is currently `main` across the workspace.

Until repository rulesets are managed centrally, this document should be treated as the per-repository source of truth for the minimum GitHub protection posture that owners are expected to apply.

## Minimum policy for `main`

Every repository should protect `main` with the following baseline rules.

Minimum expectations:

- pull requests are required before merge
- at least one approving review is required before merge
- direct pushes to `main` should be reserved for controlled automation or explicit break-glass cases
- the required validation workflows listed for the repository must be green before merge
- force-pushes to `main` should remain disabled
- branch deletion should remain disabled for `main`

For executable runtime repositories, these branch protection rules are part of the delivery safety model because automatic staging continuation depends on green service validation.

## Repository-specific required checks

The table below documents the minimum required workflow set that should be configured as blocking checks on `main`.

| Repository | Delivery branch | Minimum PR reviews | Required blocking workflow(s) | Notes |
|---|---|---|---|---|
| `ai-engine` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking because `main` drives covered staging continuation for the API and stats services |
| `api-gateway` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking before dispatch to `platform-infra` |
| `backoffice` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking before staging continuation |
| `bff-backoffice` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking before staging continuation |
| `bff-mobile` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking before staging continuation |
| `microservice-users` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking before staging continuation |
| `microservice-quizz` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking before staging continuation |
| `microservice-wordpass` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Must remain blocking before staging continuation |
| `mobile-app` | `main` | 1 | `CI` from `.github/workflows/ci.yml` | Protected like runtime code, but outside the centralized runtime-image staging chain |
| `contracts-and-schemas` | `main` | 1 | `Validate Contracts` from `.github/workflows/validate-contracts.yml` | Contract validation must block merges because downstream SDK and service consumers depend on it |
| `shared-sdk-client` | `main` | 1 | `Validate SDK Layout` and `TypeScript SDK CI` | Both workflows should be blocking because this repo publishes reusable client artifacts |
| `platform-infra` | `main` | 1 | `Validate Infra` from `.github/workflows/validate-infra.yml` | Deployment controller repository; incorrect merges affect the full delivery system |
| `observability-platform` | `main` | 1 | `Validate Observability Layout` from `.github/workflows/validate-observability.yml` | Monitoring and alert assets should not merge without validation |
| `secrets` | `main` | 1 | `Secrets Central CI` from `.github/workflows/secrets-central-ci.yml` | `Nightly Edge Smoke` is operational monitoring, not a merge-blocking branch protection check |
| `docs` | `main` | 1 | `Validate Docs` from `.github/workflows/validate-docs.yml` | Docs remain protected because cross-repository governance relies on current-state accuracy |

## Runtime repository expectations

For covered runtime repositories, owners should additionally ensure that:

- the required `CI` workflow is the workflow selected as a required status check in GitHub branch protection or rulesets
- merge is blocked when coverage, tests, lint, or compile validation fail inside that workflow
- automation that dispatches `platform-infra` remains downstream of successful validation jobs only

## Platform and documentation repository expectations

For platform, shared-code, and documentation repositories:

- the validation workflow should still be configured as required on `main`
- branch protection should still require pull requests and one approving review
- repositories with multiple validation workflows should require all workflows that protect correctness of published artifacts or governed assets

## Operational note

This document defines the required protection posture. It does not, by itself, prove that every GitHub repository already has the corresponding settings configured.

Repository owners should treat drift between this document and GitHub settings as a governance defect and correct it in the repository settings or org-level rulesets.

## Related documents

- [Repository minimum standards](./repository-minimum-standards.md)
- [GitHub governance baseline](./github-governance-baseline.md)
- [CI/CD workflow map](./cicd-workflow-map.md)
- [Deployment strategy](./deployment-strategy.md)
- [Repository standards adoption guide](../guides/repository-standards-adoption.md)