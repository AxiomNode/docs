# GitHub Actions hardening policy

> Status: 2026-04-23 — first revision shipped alongside top-risk #4 of
> [`threat-model.md`](./threat-model.md). Pairs with
> [`environments-and-secrets.md`](./environments-and-secrets.md) and
> [`secrets-rotation-policy.md`](./secrets-rotation-policy.md).

This document is the source of truth for how AxiomNode hardens its GitHub
Actions pipelines. It complements the workflow map in
[`cicd-workflow-map.md`](./cicd-workflow-map.md) and the rotation cadence in
[`secrets-rotation-policy.md`](./secrets-rotation-policy.md). The goals are:

1. Every workflow runs with the **least privilege** necessary.
2. Every CI/CD secret is a **fine-grained personal access token** or a
   **GitHub App credential**, always scoped to the smallest possible blast
   radius.
3. Every deploy to `stg` or `prod` goes through a **GitHub Environment with
   required reviewers**.
4. Every workflow change is reviewed by a CODEOWNER from the platform team.

## 1. Required GitHub Environments

GitHub Environments are configured in repo `Settings → Environments`. The
following matrix MUST be honoured:

| Repository       | Environment      | Branches allowed   | Required reviewers      | Wait timer | Secrets scoped here                                                            |
|------------------|------------------|--------------------|-------------------------|------------|--------------------------------------------------------------------------------|
| `platform-infra` | `stg`            | `main`             | 1 (platform team)       | 0          | `K3S_HOST`, `K3S_USER`, `K3S_SSH_KEY`, `GHCR_PULL_USERNAME`, `GHCR_PULL_TOKEN` |
| `platform-infra` | `prod`           | `main` (tags only) | 2 (platform + on-call)  | 5 min      | Same as `stg` plus prod-only overrides                                         |
| `platform-infra` | `build`          | `main`, `develop`  | 0 (auto)                | 0          | `CROSS_REPO_READ_TOKEN`                                                        |
| `shared-sdk-client` | `npm-publish` | tags `typescript-sdk-v*` | 1 (SDK owner)      | 0          | `NPM_TOKEN`                                                                    |
| `secrets`        | `audit`          | `main`             | 0 (auto)                | 0          | `SECRETS_CROSS_REPO_READ_TOKEN`                                                |

Notes:

- `platform-infra/.github/workflows/deploy.yaml` already declares
  `environment: ${{ inputs.environment || 'stg' }}` so the protection rules
  configured in the GitHub UI are enforced automatically once they exist.
- The `prod` reviewer set MUST include at least one operator outside of the
  person opening the deployment PR (no self-approvals).

## 2. Token inventory and minimum scopes

All CI/CD tokens are **fine-grained PATs** issued from a service account
(`axiomnode-ci`) with explicit expiration. Classic PATs with `repo` scope are
deprecated and being phased out (see migration plan in §5).

| Secret name                     | Owner repo(s)                     | Token type   | Resource owner | Repository access            | Permissions                                | Rotation |
|---------------------------------|-----------------------------------|--------------|----------------|------------------------------|--------------------------------------------|----------|
| `PLATFORM_INFRA_DISPATCH_TOKEN` | every service repo                | Fine-grained | `AxiomNode`    | `platform-infra` only        | `actions: write`                           | 180 days |
| `PLATFORM_INFRA_DISPATCH_APP_CLIENT_ID` / `PLATFORM_INFRA_DISPATCH_APP_PRIVATE_KEY` | every service repo | GitHub App var + secret | `AxiomNode` | `platform-infra` only | installation token with `actions: write`   | app-managed |
| `CROSS_REPO_READ_TOKEN`         | `platform-infra`, BFFs, services  | Fine-grained | `AxiomNode`    | All AxiomNode repos          | `contents: read`, `metadata: read`         | 180 days |
| `SECRETS_CROSS_REPO_READ_TOKEN` | `secrets`                         | Fine-grained | `AxiomNode`    | All AxiomNode repos          | `contents: read`, `metadata: read`         | 180 days |
| `GHCR_PULL_TOKEN`               | `platform-infra` (deploy.yaml)    | Fine-grained | `AxiomNode`    | `platform-infra` only        | `packages: read`                           | 180 days |
| `GHCR_PULL_USERNAME`            | `platform-infra` (deploy.yaml)    | Username     | n/a            | n/a                          | n/a (matches PAT account)                  | n/a      |
| `K3S_HOST` / `K3S_USER`         | `platform-infra` (deploy.yaml)    | String       | n/a            | n/a                          | n/a                                        | rare     |
| `K3S_SSH_KEY`                   | `platform-infra` (deploy.yaml)    | SSH keypair  | n/a            | n/a                          | sudo-less user on VPS                      | 365 days |
| `NPM_TOKEN`                     | `shared-sdk-client`               | npm token    | npmjs.org      | `@axiomnode/*` packages      | `publish` only                             | 180 days |
| `CODECOV_TOKEN`                 | every service repo                | Codecov      | codecov.io     | per-repo                     | upload coverage                            | n/a      |

Anti-patterns to remove on sight:

- `repo` scope on classic PATs (grants write to every public + private repo).
- `admin:org` on any CI token.
- A single PAT shared across `dispatch`, `read`, and `packages` purposes.
- Storing tokens at the repo level when an Environment can scope them tighter.

## 3. Workflow standards

Every `.github/workflows/*.yml` MUST satisfy the following checklist before
merge. The platform team enforces this through CODEOWNERS review.

1. **Top-level `permissions:` block** declared explicitly. Defaulting to the
   GitHub-wide defaults is forbidden.
2. **Per-job `permissions:` overrides** when a job needs less than the
   workflow-level grant (e.g. a lint job inside `build-push.yaml` should not
   inherit `packages: write`).
3. **`environment:` key** for every job that produces an externally visible
   side-effect (deploy, image push to a protected tag, npm publish).
4. **Secrets are referenced by name only**, never echoed. Multi-line secrets
   (PEM keys) MUST be written through a heredoc/`printf` and re-masked with
   `::add-mask::` to defend against multi-line redaction gaps in the runner.
5. **Cross-repo dispatches MUST validate the dispatch token** before running
   `curl`/`gh workflow run`, fail fast if it is empty, and surface the HTTP
   status in the job log. Prefer a short-lived GitHub App installation token;
   keep the fine-grained PAT only as compatibility fallback while app
   credentials are being rolled out.
6. **Pinned action versions**: third-party actions are pinned to a SHA or a
   major release; `@latest` and floating `@v*` tags on community actions are
   forbidden in deploy workflows.
7. **Concurrency groups** for any workflow that mutates shared state
   (deploys, image builds, secrets sync).

The current state of compliance is summarised in
[`cicd-workflow-map.md`](./cicd-workflow-map.md). Audits run as part of the
quarterly architecture retro
([`retrospective-process.md`](./retrospective-process.md)).

## 4. CODEOWNERS expectations

Each repository SHOULD ship a `.github/CODEOWNERS` that requires a platform
team review for workflow changes:

```text
# .github/CODEOWNERS
.github/workflows/  @AxiomNode/platform-team
.github/actions/    @AxiomNode/platform-team
```

`platform-infra`, `secrets`, and `shared-sdk-client` MUST also gate
`Dockerfile`, `kubernetes/`, and `terraform/` paths through the same team.

## 5. Migration plan: classic PATs → fine-grained

| Step | Owner          | Status (2026-04-23) |
|------|----------------|---------------------|
| 1. Issue fine-grained PAT for `PLATFORM_INFRA_DISPATCH_TOKEN` and rotate the old one. | Platform on-call | **Done** (2026-04-22) |
| 1b. Roll out optional GitHub App credentials (`PLATFORM_INFRA_DISPATCH_APP_CLIENT_ID` + `PLATFORM_INFRA_DISPATCH_APP_PRIVATE_KEY`) to every service repo so dispatch can stop depending on PATs. | Platform on-call | **In progress** — workflow fallback shipped 2026-04-26 |
| 2. Issue fine-grained PAT for `CROSS_REPO_READ_TOKEN`.                                | Platform on-call | **Done** (2026-04-22) |
| 3. Issue fine-grained PAT for `SECRETS_CROSS_REPO_READ_TOKEN`.                        | Platform on-call | **Done** (2026-04-22) |
| 4. Replace classic `GHCR_PULL_TOKEN` with a fine-grained PAT scoped to `packages: read` only. | Platform on-call | **Pending** — depends on next rotation window |
| 5. Configure GitHub Environments (`stg`, `prod`, `build`, `npm-publish`, `audit`) with the reviewer matrix in §1. | Platform on-call | **In progress** — workflow keys shipped 2026-04-26; reviewer setup still UI-only |
| 6. Add `.github/CODEOWNERS` with `.github/workflows/` rule across every active repo. | Platform on-call | **Done** (2026-04-26) |
| 7. Annotate every workflow's `permissions:` block with a comment explaining each grant. | Platform team    | In progress (`platform-infra/build-push.yaml`, `deploy.yaml` done 2026-04-23) |

The remaining steps are tracked alongside top-risk #4 in
[`threat-model.md`](./threat-model.md).

## 6. Operational runbook: onboarding a new repo

When a new repository joins the AxiomNode org:

1. Copy the workflow template from an existing service (e.g. `bff-backoffice`).
2. Add the secrets it actually needs — never paste in a superset.
3. Configure Environments in the GitHub UI according to §1.
4. Add `.github/CODEOWNERS` with the platform team gating workflows.
5. Run the secrets audit job in `secrets/` against the new repo to validate
   nothing was hardcoded.
6. Update [`cicd-workflow-map.md`](./cicd-workflow-map.md) with the new
   dispatch arrows.

## 7. Out of scope (future work)

- OIDC federation between GitHub Actions and the GHCR / VPS so we can drop
  the static `GHCR_PULL_TOKEN` and `K3S_SSH_KEY` entirely.
- Replacing `peter-evans/repository-dispatch`-style raw `curl` with reusable
  workflows that inherit secrets via `secrets: inherit`.
- Sigstore / cosign signature on every image pushed to GHCR.
