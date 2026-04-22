# Repository Minimum Standards

Last updated: 2026-04-22.

## Intent

This document defines the minimum engineering standards that every AxiomNode repository must satisfy.

These standards are intentionally small, enforceable, and cross-repository. They are meant to reduce variance between repositories and make quality gates predictable.

## Scope

This standard applies to all repositories in the AxiomNode workspace, but some rules are stricter for executable runtime repositories than for documentation-only, validation-only, or packaging-only repositories.

For this document, the main repository classes are:

- executable runtime repositories: services, frontends, BFFs, and AI runtime APIs
- platform or shared-code repositories: SDKs, schemas, infrastructure, observability automation, secrets automation
- documentation-only repositories

## Minimum standards

### 1. Clear architecture boundary

Every executable repository must follow an explicit architectural structure.

Minimum rule:

- business logic must not be mixed arbitrarily with transport, persistence, or framework glue
- ownership boundaries must be visible in the folder structure and documented locally
- dependency direction should prefer inward-facing domain or application code over framework-driven coupling

Accepted implementation styles include:

- clean architecture
- layered architecture with equivalent separation of concerns
- other explicit architectures with the same boundary discipline

The required outcome is not one folder name. The required outcome is architectural separation that is understandable, reviewable, and enforceable.

### 2. Coverage gate at or above 90%

Executable repositories must enforce automated test coverage of at least 90% using the richest reliable metrics supported by their stack.

Rules:

- Vitest repositories must enforce 90% for lines, statements, functions, and branches
- Python repositories using coverage.py must enforce the current 90% total gate with branch instrumentation enabled
- coverage must be a CI failure condition, not an informational report only

This document complements, but does not replace, the detailed policy in [Test coverage policy](./test-coverage-policy.md).

### 3. Documentation in English

Repository documentation must be written in English.

Minimum scope:

- root README
- local architecture notes
- operational runbooks
- CI or release notes that describe repository behavior

Short code comments may use technical shorthand, but project-facing documentation must remain English-first so it is consistent across the workspace.

### 4. Mandatory CI validation on every change to the delivery branch

Every executable repository must have a mandatory workflow that validates the repository before delivery continues.

Minimum expectations:

- the main repository CI workflow must run automatically on pushes and pull requests for the active delivery branch
- all required quality gates must pass before delivery can continue
- a failing test, coverage gate, lint gate, or compile gate must stop the delivery chain

For runtime repositories, this workflow is required to include at least:

- dependency installation
- build or compile validation when relevant
- automated tests
- mandatory coverage enforcement

### 5. Automatic staging deployment for covered runtime repositories

Deployable runtime repositories must automatically continue to staging after a successful update to `main`.

Minimum rule:

- if the repository is part of the covered staging deployment set, a successful change on `main` must automatically trigger the centralized image build and staging rollout chain

This rule applies to covered runtime repositories only. It does not automatically apply to:

- documentation-only repositories
- validation-only repositories
- repositories that do not publish a runtime image
- manually controlled production-only assets

The effective current staging deployment chain is documented in [Deployment strategy](./deployment-strategy.md) and [CI/CD workflow map](./cicd-workflow-map.md).

### 6. Required workflows must be blocking, not optional

Repository workflows that validate correctness must be treated as required gates.

Minimum expectations:

- repository owners must not rely on manual judgment to ignore a red validation workflow
- coverage verification must be part of the required workflow set when the repository is executable
- automated deployment dispatch must depend on successful validation jobs

In practical terms, the repository must behave as if "all mandatory checks must be green" before delivery continues.

## Repository-class expectations

### Executable runtime repositories

These repositories are expected to satisfy the full standard:

- explicit architecture boundaries
- English documentation
- mandatory CI validation
- mandatory automated tests
- mandatory coverage gate at or above 90%
- automatic staging continuation after successful `main` updates when included in the covered deployment chain

### Platform and shared-code repositories

These repositories must still satisfy the documentation and CI discipline rules, but deployment and coverage enforcement may differ depending on their actual execution surface.

Minimum expectations:

- English documentation
- explicit repository ownership and structure
- required validation workflows
- executable tests and coverage gates whenever the repository exposes a meaningful testable runtime or library surface

### Documentation-only repositories

These repositories are not required to maintain a 90% executable coverage gate or automatic staging deployment, but they still must satisfy minimum governance quality:

- English documentation
- required validation workflows when documentation validation exists
- current-state accuracy
- valid links and maintainable structure

## Current repository standards matrix

This matrix captures the current expected governance position of each repository in the workspace.

Legend:

- `Full`: the repository is expected to satisfy the full executable-repository standard
- `Partial`: the repository is expected to satisfy documentation and CI discipline, but some runtime-specific rules do not apply directly
- `N/A`: the rule is not currently applicable in the same way as deployable runtime services

| Repository | Class | Architecture standard | English docs required | Required CI workflow expected | 90% coverage gate expected | Automatic `stg` continuation expected | Current verified status | Current note |
|---|---|---|---|---|---|---|---|---|
| `ai-engine` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository; current automatic chain covers API and stats runtime images |
| `api-gateway` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository |
| `backoffice` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository |
| `bff-backoffice` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository |
| `bff-mobile` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository |
| `microservice-users` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository |
| `microservice-quizz` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository |
| `microservice-wordpass` | executable runtime | Full | Yes | Yes | Yes | Yes | Verified compliant on 2026-04-22 for current enforced gate and staging-chain inclusion | Covered runtime repository |
| `mobile-app` | executable runtime | Full | Yes | Yes | Coverage instrumentation still required to be explicit per mobile toolchain | No | Partially verified: CI exists, but no equivalent automatic `stg` runtime chain and no workspace-level 90% gate currently verified | Validated in its own CI, but not deployed to k3s staging through the central runtime-image chain |
| `contracts-and-schemas` | platform/shared | Partial | Yes | Yes | Only where executable validation surfaces exist | No | Partially verified: validation workflow exists; runtime-specific rules intentionally out of scope | Validation repository, not a runtime image publisher |
| `shared-sdk-client` | platform/shared | Partial | Yes | Yes | Only where executable library coverage is instrumented and enforced | No | Partially verified: CI exists; runtime auto-deploy is intentionally out of scope | Shared library repository, not a staging runtime deployment source |
| `platform-infra` | platform/shared | Partial | Yes | Yes | Only where executable testable code exists | No | Partially verified: validation workflows exist; repository is deployment controller, not deployment target | Owns packaging and deployment policy rather than being auto-deployed as an app |
| `observability-platform` | platform/shared | Partial | Yes | Yes | Only where executable validation surfaces exist | No | Partially verified: validation workflow exists; automatic runtime staging continuation is intentionally out of scope | Monitoring assets repository, not a covered runtime image source |
| `secrets` | platform/shared | Partial | Yes | Yes | N/A for non-runtime governance workflows unless an executable surface is introduced | No | Partially verified: validation workflows exist; runtime deployment and coverage gate do not apply in the same way | Governance and automation repository, not a covered runtime image source |
| `docs` | documentation-only | Partial | Yes | Yes | N/A | No | Verified for documentation-class expectations with markdown validation workflow | Documentation repository with markdown validation workflow |

## Mandatory pull request checklist

Every executable repository should expose a blocking pull request checklist equivalent to the following minimum gate.

Recommended baseline checklist:

- [ ] Architecture boundaries remain explicit and framework glue has not leaked into domain or application logic
- [ ] Repository-facing documentation affected by the change is updated in English
- [ ] Build, compile, or packaging validation is green when applicable
- [ ] Automated tests are green
- [ ] Coverage remains at or above the enforced repository threshold
- [ ] CI workflows for the repository remain blocking, not advisory
- [ ] If the repository is in the covered staging chain, successful `main` delivery still continues automatically to `stg`
- [ ] Cross-repository contract, SDK, or deployment implications are documented when affected

Use this checklist as:

- a repository PR template section
- a protected-branch review checklist
- a release-readiness checklist for service owners

## Mandatory new repository bootstrap checklist

Any new repository added to the workspace should be evaluated against this bootstrap checklist before being treated as production-grade.

Recommended bootstrap checklist:

- [ ] The repository role is clearly classified: executable runtime, platform/shared, or documentation-only
- [ ] The repository has an explicit architecture and dependency boundary model
- [ ] The root README is written in English and explains responsibility, local workflow, and ownership boundaries
- [ ] A required CI workflow exists and runs automatically on pull requests and the delivery branch
- [ ] Tests exist for the executable surface of the repository
- [ ] A mandatory coverage gate is configured when the repository has an executable code surface that supports it reliably
- [ ] Runtime repositories document how they enter, or why they do not enter, the centralized automatic staging deployment chain
- [ ] Repository-specific release constraints, smoke checks, and external dependencies are documented
- [ ] Cross-repository contracts and SDK dependencies are documented when applicable

## Repositories outside the automatic staging deployment chain

The following repositories are intentionally outside the current automatic `stg` runtime deployment chain, and the reason should stay explicit in governance docs.

| Repository | Why it is outside the current automatic `stg` chain |
|---|---|
| `mobile-app` | It has its own CI, but it does not publish a GHCR runtime image deployed to the k3s staging cluster through `platform-infra` |
| `contracts-and-schemas` | It is a contracts source-of-truth repository, not a runtime service |
| `shared-sdk-client` | It publishes shared SDK artifacts, not a staging runtime service |
| `platform-infra` | It owns packaging and deployment policy; it is the delivery controller, not a target runtime service |
| `observability-platform` | It contains monitoring stack assets and validation, not a covered application runtime image in the service deployment chain |
| `secrets` | It governs secrets and related automation, not a covered runtime service |
| `docs` | It is a documentation repository with validation workflows only |

Operational note:

- the external llama runtime can also be outside the current automatic staging chain when staging runs in split-AI mode; this is a runtime-topology concern rather than a standalone repository classification

## Review checklist

Use this checklist when creating or auditing a repository:

1. Is the architecture explicit and separated from framework glue?
2. Is repository-facing documentation written in English?
3. Does CI run automatically on the delivery branch and on pull requests?
4. Do required workflows block delivery when tests or build validation fail?
5. Is coverage enforced at 90% or higher when the repository is executable?
6. If the repository is deployable to staging, does a green `main` update continue automatically through the centralized staging deployment chain?

## Related documents

- [Test coverage policy](./test-coverage-policy.md)
- [Branch protection policy](./branch-protection-policy.md)
- [Deployment strategy](./deployment-strategy.md)
- [CI/CD workflow map](./cicd-workflow-map.md)
- [Migration guide for existing repositories](../guides/migration-existing-repos.md)
- [Repository standards adoption guide](../guides/repository-standards-adoption.md)