# AxiomNode Workspace Audit

Last updated: 2026-04-13.

## Scope

This audit covers all repositories currently present in the local AxiomNode workspace:

- `ai-engine`
- `api-gateway`
- `backoffice`
- `bff-backoffice`
- `bff-mobile`
- `contracts-and-schemas`
- `docs`
- `microservice-quizz`
- `microservice-users`
- `microservice-wordpass`
- `mobile-app`
- `observability-platform`
- `platform-infra`
- `secrets`
- `shared-sdk-client`

## Method

- Reviewed repository structure, README coverage, workflow presence, and test directory presence.
- Sampled CI/CD workflows, deployment scripts, staging manifests, and selected package/runtime definitions.
- Verified live staging cluster state on `sebss@amksandbox.cloud` through SSH and `k3s kubectl`.
- Focused findings on efficacy, efficiency, security, clean code, documentation accuracy, and code comment quality.

This is a snapshot audit, not a full line-by-line review of every source file.

## Executive Summary

The workspace shows a strong architectural split and generally consistent repository hygiene: every repository has a `README.md`, almost every active runtime repository has a CI workflow, and service boundaries are clearly documented.

The main issues are not random code-level defects. They are concentrated in four areas:

1. staging deployment hardening is incomplete and currently blocks several pods from starting,
2. CI quality gates are inconsistent across repository types,
3. documentation is generally good but contains important workflow drift,
4. frontend/mobile validation remains materially weaker than backend validation.

## Key Findings

### 1. Critical: staging is not operationally healthy due to security-context and image-user mismatch

Affected repos:

- `platform-infra`
- `ai-engine`
- `api-gateway`
- `bff-mobile`
- `bff-backoffice`
- `microservice-quizz`
- `microservice-users`
- `microservice-wordpass`

Evidence:

- Live `k3s` inspection on `sebss@amksandbox.cloud` showed multiple pods in `CreateContainerConfigError`.
- `ai-engine-api` failed with: `image has non-numeric user (appuser), cannot verify user is non-root`.
- `api-gateway` failed with: `image has non-numeric user (node), cannot verify user is non-root`.
- `ai-engine-llama` failed with: `image will run as root` while the manifest forces `runAsNonRoot: true`.

Why it matters:

- Staging cannot be treated as a trustworthy pre-release environment while core services are unable to start under the declared hardening policy.
- This is a deployment correctness issue and a security posture issue at the same time.

Recommended action:

1. Keep explicit numeric `runAsUser` and `runAsGroup` in Kubernetes manifests for all images that do not already declare numeric users.
2. Standardize Dockerfiles to use explicit numeric users instead of symbolic users where possible.
3. Decide and document the intended runtime model for `ai-engine-llama`: either run it with an explicit non-root UID or explicitly scope an exception with justification.

### 2. High: deployment policy documentation is contradictory inside `platform-infra`

Affected repos:

- `platform-infra`
- `docs`

Evidence:

- `platform-infra/AGENTS.md` says automatic deploy currently targets `dev`.
- `platform-infra/README.md`, `platform-infra/kubernetes/README.md`, and `platform-infra/environments/stg/README.md` all describe `stg` as the active automatic target.
- The current GitHub workflow configuration also targets `stg`.

Why it matters:

- Operators and contributors cannot rely on the repo-local instructions for deployment behavior.
- This directly increases release risk because deploy expectations differ depending on which file is read.

Recommended action:

1. Update `platform-infra/AGENTS.md` so it matches the current workflow behavior.
2. Add one canonical deployment-policy reference in `docs/operations` and link to it from `platform-infra`.

### 3. High: `mobile-app` has no CI workflow at all

Affected repos:

- `mobile-app`

Evidence:

- `mobile-app/README.md` explicitly states there is no dedicated workflow in `.github/workflows`.
- The repository has no GitHub Actions workflow directory in this workspace snapshot.

Why it matters:

- The mobile client is outside the backend CI safety net.
- Build failures, lint regressions, dependency issues, and contract drift can land on `main` without automated detection.

Recommended action:

1. Add a baseline CI workflow for build, tests, lint, and dependency checks.
2. Add contract validation against the current edge API surface.

### 4. High: `backoffice` CI only builds, but the repository already contains test tooling and documentation expects more

Affected repos:

- `backoffice`
- `docs`

Evidence:

- `backoffice/package.json` includes Vitest scripts such as `test`, `test:run`, and `test:coverage`.
- `backoffice/.github/workflows/ci.yml` runs `npm run build` only.
- `docs/guides/backoffice-react-development.md` describes a testing baseline that includes unit, integration, and E2E coverage expectations.

Why it matters:

- The repository communicates a stronger validation story than the CI pipeline actually enforces.
- Frontend regressions can pass CI as long as the bundle builds.

Recommended action:

1. Add `npm run test:run` to CI immediately.
2. Add a `typecheck` script and run it explicitly.
3. Reword the documentation to distinguish current state from target-state testing strategy.

### 5. High: TypeScript services do not enforce coverage thresholds, while Python does

Affected repos:

- `api-gateway`
- `bff-mobile`
- `bff-backoffice`
- `microservice-quizz`
- `microservice-users`
- `microservice-wordpass`
- `backoffice`

Evidence:

- The TypeScript runtime repositories include tests, but no consistent coverage threshold enforcement was found in sampled workflows.
- `ai-engine` explicitly enforces a coverage floor in CI, creating an asymmetric quality gate across the platform.

Why it matters:

- Coverage expectations are inconsistent across stacks.
- Backend TypeScript regressions have a weaker measurable quality barrier than the Python AI service.

Recommended action:

1. Add coverage reporting and a minimum threshold to all active TypeScript service repos.
2. Start with a realistic baseline and ratchet upward instead of leaving coverage ungated.

### 6. Medium: mutable image-tag policy is still partially allowed by validation

Affected repos:

- `platform-infra`

Evidence:

- `platform-infra/.github/workflows/validate-infra.yml` blocks `:latest` but does not block mutable variants such as `stg-latest` or `dev-latest`.
- The staging deployment model still relies on mutable tags in practice.

Why it matters:

- Mutable tags weaken traceability and rollback confidence.
- With `imagePullPolicy: Always`, repeated restarts can pull different artifacts for the same tag.

Recommended action:

1. Move staging references toward immutable tags or commit-based tags.
2. Extend infra validation so mutable `-latest` variants are treated as violations too.

### 7. Medium: nightly edge smoke intentionally disables strict Firebase auth checks

Affected repos:

- `secrets`
- `microservice-users`

Evidence:

- `secrets/scripts/run-nightly-edge-smoke.sh` forces `FIREBASE_STRICT_AUTH=false` in the local smoke environment before booting `microservice-users`.

Why it matters:

- This is understandable for dev-smoke resilience, but it also means one important class of staging/prod authentication misconfiguration is not exercised by that smoke path.

Recommended action:

1. Keep the permissive smoke path for edge integration if needed.
2. Add a second stricter verification path for environments intended to validate real Firebase credential behavior.

### 8. Medium: documentation quality is generally good, but `docs` has no automated validation workflow

Affected repos:

- `docs`

Evidence:

- The documentation repository is well-organized and repository mapping is clear.
- The repo currently has no workflow for link validation, stale-page checks, or consistency guards.

Why it matters:

- Documentation drift is already visible in deployment-policy guidance.
- Without automated checks, doc quality depends entirely on manual discipline.

Recommended action:

1. Add at least a lightweight link and markdown validation workflow.
2. Add a small checklist for “current-state accuracy” in operational docs.

## Code Comments Assessment

Comments are generally purposeful where they exist.

- Service code in `ai-engine` and several TypeScript backends uses concise module-level comments and avoids noisy line-by-line narration.
- Infrastructure YAML and shell scripts rely more on external docs than inline rationale. That keeps files shorter, but it also makes operational intent harder to recover during incidents.

Overall assessment:

- Application-code commenting style: good.
- Operational manifest/commenting style: adequate but under-explained for high-risk deployment behavior.

## Repository Health Snapshot

| Repository | Snapshot |
|---|---|
| `ai-engine` | Strong code/test discipline; staging runtime still blocked by image-user mismatch. |
| `api-gateway` | Clean structure and tests present; staging manifest hardening was incomplete. |
| `backoffice` | Buildable and test-capable, but CI under-validates current frontend quality. |
| `bff-backoffice` | Structurally sound; same coverage and runtime-user gaps as other Node services. |
| `bff-mobile` | Structurally sound; same coverage and runtime-user gaps as other Node services. |
| `contracts-and-schemas` | Lean and focused; low complexity, but limited independent validation surface. |
| `docs` | Good central architecture and ops coverage; lacks automated validation. |
| `microservice-quizz` | Healthy service shape; CI/runtime improvements were recently required and should be stabilized. |
| `microservice-users` | Health improved materially; smoke/auth trade-offs should be documented more explicitly. |
| `microservice-wordpass` | Similar to other game services; sound structure, moderate validation depth. |
| `mobile-app` | Highest process risk due to complete absence of CI automation. |
| `observability-platform` | Small footprint; low visible validation depth in current workspace snapshot. |
| `platform-infra` | Operationally central and mostly well-structured; currently the highest leverage repo for security and deployment reliability fixes. |
| `secrets` | Strong centralized role and useful smoke coverage; some checks are intentionally permissive. |
| `shared-sdk-client` | Important contract backbone; quality depends on downstream adoption discipline. |

## Priority Actions

### Immediate

1. Recover staging startup by aligning Kubernetes security context with real image runtime users.
2. Fix the `platform-infra` deployment-policy documentation drift.
3. Add CI for `mobile-app`.

### Next

1. Add TypeScript coverage thresholds across service repos.
2. Upgrade `backoffice` CI from build-only to build-plus-test-plus-typecheck.
3. Harden mutable image-tag validation.

### Later

1. Add docs validation workflow.
2. Add stricter auth-sensitive smoke coverage for Firebase-backed flows.
3. Standardize Dockerfiles on explicit numeric runtime users.