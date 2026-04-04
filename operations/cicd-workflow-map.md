# CI/CD Workflow Map

Last updated: 2026-04-05.

This document maps how CI/CD currently works across AxiomNode repositories.

## 1. Service repository CI

Repositories with runtime services (for example `api-gateway`, `bff-mobile`, `microservice-quizz`, `ai-engine`) run their own `.github/workflows/ci.yml`.

Typical checks include:

- dependency install
- build
- tests
- lint
- additional audits/smoke checks (repo-specific)

## 2. Cross-repo dispatch to platform-infra

On push to `main`, service repository CI triggers:

- `platform-infra/.github/workflows/build-push.yaml`

Dispatch uses `PLATFORM_INFRA_DISPATCH_TOKEN` from the service repository.

## 3. Platform image build and publish

`platform-infra` workflow `build-push.yaml`:

- validates cross-repo read token access
- checks out source repositories
- builds service-specific images
- pushes images to GHCR

Tag behavior:

- non-main branches: commit SHA + `dev`
- `main`: commit SHA + `dev` + `stg` + `latest`

## 4. Deployment workflow

`platform-infra/.github/workflows/deploy.yaml` runs:

- after successful completion of `build-push.yaml` on `main`
- or by manual dispatch

Current automatic rollout policy:

- auto deploy target is `dev` only
- deployment applies Kubernetes overlays in `axiomnode-dev`
- rollout restart is executed to refresh mutable tags

## 5. Secrets and governance workflows

`secrets` repository workflows:

- `secrets-central-ci.yml`: sync/placeholder validation and cross-repo secret audits
- `nightly-edge-smoke.yml`: nightly dev smoke for edge integration path

## 6. Related required secrets

- In service repos: `PLATFORM_INFRA_DISPATCH_TOKEN`
- In `platform-infra`: `CROSS_REPO_READ_TOKEN`, `GHCR_PULL_USERNAME`, `GHCR_PULL_TOKEN`, `K3S_*`
- In `secrets`: `SECRETS_CROSS_REPO_READ_TOKEN`
