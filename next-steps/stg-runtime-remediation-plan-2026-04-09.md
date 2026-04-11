# Staging Runtime Remediation Plan

Last updated: 2026-04-09.

## Goal

Recover staging to the intended AxiomNode operating model and close the main gaps detected in the live audit:

- restore functional correctness for mobile and AI flows,
- reduce operational fragility,
- improve latency stability and resource efficiency,
- align runtime security posture across all deployed services.

## Scope

This plan covers the services currently deployed through `platform-infra` in `axiomnode-stg`:

- `api-gateway`
- `backoffice`
- `bff-mobile`
- `bff-backoffice`
- `microservice-quizz`
- `microservice-wordpass`
- `microservice-users`
- `ai-engine-api`
- `ai-engine-stats`
- `ai-engine-llama`
- staging Kubernetes manifests, ingress exposure, and runtime secrets/config wiring

## Current runtime findings that drive this plan

1. `api-gateway` in staging is running an old bundle that still requires `EDGE_API_TOKEN` on mobile routes.
2. `ai-engine-api` and `ai-engine-stats` are in `CrashLoopBackOff`, with Uvicorn parent alive and worker children dying before opening the health port.
3. Node memory is too tight for the current shape of the stack, mainly due to `ai-engine-llama` footprint.
4. Most deployments still run with the default service account, implicit token mount, and no meaningful pod/container hardening.
5. Public edge health is acceptable but unstable, with visible latency spikes even under low observed load.
6. Host listening sockets show a broader attack surface than the intended public product surface.

## Execution status on 2026-04-09

Already completed in source and infra:

1. `api-gateway` source was re-verified: mobile random routes are already public in source and covered by tests.
2. The staging mismatch was traced to runtime drift: the deployed gateway bundle/image is older than current source.
3. `ai-engine` startup was validated at import level locally; both FastAPI apps load correctly in the workspace Python environment.
4. `ai-engine/src/Dockerfile` was corrected so `runtime-api` and `runtime-stats` honor `PORT` and `UVICORN_WORKERS` from runtime env instead of hardcoded worker counts.
5. `platform-infra` staging overlay now forces `ai-engine-api` into a safer recovery shape for staging with `replicas: 1` and `UVICORN_WORKERS=1`.
6. Relevant `ai-engine` tests passed locally for the affected area.

Additional runtime findings on 2026-04-10:

1. The user confirmed a VPS-side issue with duplicate processes for two game servers and resolved that directly on the host.
2. The centralized `platform-infra` deploy workflow did not fail on manifest validation or `kubectl apply`; it failed later with an SSH `Broken pipe` during the long rollout/restart phase.
3. Public staging still exposed the old gateway auth behavior after the failed deploy attempt: `/v1/mobile/games/quiz/random` and `/v1/mobile/games/wordpass/random` still returned `401`, while `/v1/backoffice/services` also returned `401` as expected for a protected route.
4. `platform-infra/.github/workflows/deploy.yaml` was hardened so rollout status and replica verification use one SSH session per deployment instead of one long-lived session for the full namespace, and staging smoke checks now validate gateway auth split automatically.

Still required to close staging:

1. Build and publish fresh `:stg` images for `ai-engine-api`, `ai-engine-stats`, and `api-gateway` from the current repo state.
2. Deploy the refreshed staging manifests from `platform-infra`.
3. Verify rollout health and confirm gateway mobile routes are public again in the running environment.

Current delta:

1. The required images were already rebuilt successfully in CI.
2. The remaining open item is a successful staging deploy completion plus post-deploy public verification.
3. The latest deploy no longer died on `ai-engine-api` startup probes; it now fails later because `ai-engine-api` rollout does not converge cleanly in staging, leaving an old pod in `ImagePullBackOff` while a newer replica is still being created.
4. A staging-only mitigation is being applied to move `ai-engine-api` to a simpler `Recreate` strategy so repeated deploys do not stack overlapping old/new replicas on this single-replica heavy service.

## Delivery principles

1. Fix runtime drift before tuning performance.
2. Prefer root-cause changes in source repos plus `platform-infra`, not ad hoc VPS-only hotfixes.
3. Each phase must end with explicit smoke, latency, and security validation.
4. Update documentation in the same change window as runtime behavior changes.

## Workstream A: Functional recovery

### A1. Restore public mobile routes in `api-gateway`

Problem:
Staging is serving a gateway image/bundle that still protects mobile read/generate routes with `EDGE_API_TOKEN`, which breaks the intended mobile access model.

Actions:

1. Done: confirm the current source of truth in `api-gateway` for mobile route auth policy.
2. Pending: rebuild and redeploy the correct `api-gateway:stg` image from the repo state that removes edge auth from mobile routes.
3. Pending: verify that the deployed bundle no longer contains `checkEdgeAuth(... EDGE_API_TOKEN)` on:
   - `/v1/mobile/games/quiz/random`
   - `/v1/mobile/games/wordpass/random`
   - `/v1/mobile/games/quiz/generate`
   - `/v1/mobile/games/wordpass/generate`
4. Done in source/tests: keep backoffice routes protected and verify that the auth split is intentional and tested.
5. Pending: add a release-verification check in CI or deployment documentation that greps the deployed bundle for critical route-policy markers.

Acceptance criteria:

- `GET /v1/mobile/games/quiz/random?language=es` returns `200` without `Authorization`.
- `GET /v1/mobile/games/wordpass/random?language=es` returns `200` without `Authorization`.
- Backoffice routes still return `401` without valid credentials.
- Staging bundle inspection matches the intended source behavior.

Impacted repos:

- `api-gateway`
- `platform-infra`
- `docs`

### A2. Recover `ai-engine-api` and `ai-engine-stats`

Problem:
Both AI services exit before becoming ready. Secrets are present, so this looks like process/image startup failure rather than missing runtime keys.

Actions:

1. Partially done: reproduce the startup path locally as far as available tooling allows; app imports are healthy, but Docker daemon was not available for full local container reproduction.
2. Done: check the image entrypoint/CMD and worker model in `ai-engine` for mismatch between baked `--workers` flags and runtime env knobs such as `UVICORN_WORKERS`.
3. Done for staging config: simplify staging startup to single-worker mode for `ai-engine-api`; `ai-engine-stats` already runs with single-worker intent.
4. Deferred unless redeploy still fails: add explicit startup logging around app creation, settings resolution, model/embedder initialization, and FastAPI lifespan start.
5. Deferred unless redeploy still fails: if the root cause is import or model-init failure, make it fail loudly with traceback instead of silent worker death.
6. Done: fix the Dockerfile and deployment contract so worker count is controlled in one place only.
7. Pending: re-run the AI recovery smoke documented in operations after redeploy.

Acceptance criteria:

- `ai-engine-api` rollout reaches `2/2` available.
- `ai-engine-stats` rollout reaches `1/1` available.
- `/health` returns `200` for both services.
- `ai-engine-stats /stats` and `/stats/history` return `200` with valid AI bridge/API key.
- Backoffice aggregated AI endpoints return `200` again.
- No repeated `Child process died` loop remains in logs.

Impacted repos:

- `ai-engine`
- `platform-infra`
- `docs`

## Workstream B: Performance and efficiency

### B1. Reduce staging memory pressure

Problem:
The node runs near 89% RAM with low CPU use, leaving too little buffer for spikes and restarts.

Actions:

1. Recalculate the real memory envelope for `ai-engine-llama`, `ai-engine-api`, `ai-engine-stats`, and Redis under staging load.
2. Revisit llama model size, quantization, context window, batch size, and parallelism to fit the staging objective rather than production ambitions.
3. Lower reserved/limited memory where measurements prove it is safe.
4. If necessary, separate AI workloads from the rest of the stack by node role, namespace strategy, or host sizing.
5. Add a documented capacity budget for staging in `docs/operations`.

Acceptance criteria:

- Sustained node memory baseline under normal staging load stays below 75%.
- AI service restarts do not trigger collateral instability.
- Capacity settings are documented and reproducible.

Impacted repos:

- `ai-engine`
- `platform-infra`
- `docs`

### B2. Stabilize public edge latency

Problem:
Gateway and backoffice are reachable, but latency has noticeable long-tail spikes for a low-load environment.

Actions:

1. Add a repeatable latency benchmark for public hosts and key internal hops.
2. Measure latency separately for:
   - public ingress/TLS,
   - gateway routing,
   - BFF orchestration,
   - domain-service response.
3. Review `api-gateway` and BFF timeout values, connection reuse, and upstream retry/circuit-breaker behavior under current staging topology.
4. Check whether `ai-engine` failures are causing extra timeout overhead or slow-path retries in upstream callers.
5. Tune probes, timeouts, and logging noise so health traffic does not distort baseline behavior.

Acceptance criteria:

- `gateway /health` p95 stays below 400 ms in repeated 10-run samples.
- `backoffice /` p95 stays below 350 ms in repeated 10-run samples.
- Mobile random endpoints stay below agreed p95 target once gateway drift is fixed.

Impacted repos:

- `api-gateway`
- `bff-mobile`
- `bff-backoffice`
- `platform-infra`
- `docs`

### B3. Close remaining efficiency debt already identified

Actions:

1. Evaluate replacing Zod-heavy edge validation paths if profiling proves it matters in hot routes.
2. Investigate race-condition risk areas previously left for analysis.
3. Review `BaseHTTPMiddleware` overhead in `ai-engine` and replace with lower-cost middleware when justified.
4. Reduce backoffice bundle weight with code splitting where runtime measurements show meaningful payoff.
5. Reassess lower-priority persistence and pagination optimizations only after the critical runtime issues are green.

Acceptance criteria:

- Each item is either implemented with evidence, or explicitly closed as not worth the added complexity.

Impacted repos:

- `api-gateway`
- `ai-engine`
- `backoffice`
- `docs`

## Workstream C: Security hardening

### C1. Align Kubernetes security posture across all services

Problem:
Most workloads still run without the baseline hardening already applied to `backoffice`.

Actions:

1. Apply a common baseline to all deployments:
   - `automountServiceAccountToken: false`
   - dedicated service account only when required
   - `runAsNonRoot: true`
   - explicit non-root UID where compatible
   - `allowPrivilegeEscalation: false`
   - `seccompProfile: RuntimeDefault`
   - drop Linux capabilities where possible
   - read-only root filesystem where practical
2. Validate each container image for non-root compatibility before rollout.
3. Add policy checks in infra validation to prevent regressions.

Acceptance criteria:

- All stateless app deployments in staging pass the baseline hardening checklist.
- Default service account is no longer mounted into application pods unless explicitly justified.
- CI/validation fails when a new deployment omits required hardening fields.

Impacted repos:

- `platform-infra`
- service repos as needed for image compatibility
- `docs`

### C2. Reduce external attack surface

Problem:
The host is listening on more ports than the intended public product surface.

Actions:

1. Inventory which listening ports are expected, internal-only, admin-only, or legacy.
2. Validate the effective firewall and cloud-network exposure for each listening socket.
3. Close or restrict anything not required for staging operations.
4. Document the allowed exposure model for staging.

Acceptance criteria:

- Only explicitly approved public ports remain reachable from the internet.
- Admin and cluster-control ports are restricted to trusted paths only.
- Exposure policy is documented and reviewed.

Impacted repos:

- `platform-infra`
- VPS host configuration
- `docs`

### C3. Add runtime drift detection for critical security and auth behavior

Problem:
Recent incidents were caused by runtime behavior diverging from merged source code and documented policy.

Actions:

1. Add post-deploy smoke checks for auth model split:
   - mobile routes public,
   - backoffice routes protected,
   - AI monitoring routes protected with bridge/API key.
2. Add deployment validation that inspects the running image or deployed bundle for critical route-policy markers.
3. Document a release checklist for auth-sensitive changes.

Acceptance criteria:

- A wrong gateway auth policy is detected automatically after deployment.
- AI auth regressions are caught before being discovered through backoffice failures.

Impacted repos:

- `api-gateway`
- `bff-backoffice`
- `platform-infra`
- `docs`

## Workstream D: Operability and documentation

### D1. Make recovery paths explicit and current

Actions:

1. Update the AI recovery runbook to cover staging, not only dev.
2. Add a staging verification guide for:
   - gateway/mobile auth split,
   - backoffice aggregated observability,
   - AI service recovery,
   - bundle verification after release.
3. Capture the staging rollout checks that proved necessary in this audit.

Acceptance criteria:

- An engineer can reproduce verification and recovery steps from docs alone.

Impacted repos:

- `docs`
- `platform-infra`

### D2. Add a single service-level scorecard for staging

Actions:

1. Maintain a simple table with each service, responsibility, status, key dependencies, and validation endpoints.
2. Track the service state through the remediation phases.

Acceptance criteria:

- The team can tell in one page which services are green, degraded, or blocked and why.

Impacted repos:

- `docs`

## Recommended execution order

### Phase 0: Immediate containment

1. Build and publish fresh staging images for `ai-engine-api`, `ai-engine-stats`, and `api-gateway`.
2. Deploy the updated `platform-infra` staging overlay.
3. Verify `ai-engine-api`, `ai-engine-stats`, and gateway mobile routes in the running environment.

### Phase 1: Runtime stabilization

1. Reduce staging memory pressure.
2. Stabilize public edge latency.
3. Confirm no hidden runtime drift remains in gateway, BFFs, and AI paths.

### Phase 2: Security baseline

1. Roll out Kubernetes hardening to all services.
2. Reduce external exposure.
3. Add CI and deploy-time drift/security checks.

### Phase 3: Structural efficiency

1. Close remaining performance debt items that still matter after stabilization.
2. Consolidate documentation and scorecards.

## Verification matrix

The plan is complete only when all of the following are true:

1. Public mobile routes work without edge token and backoffice stays protected.
2. `ai-engine-api` and `ai-engine-stats` are stable, healthy, and observable.
3. Node memory baseline is within the defined operating budget.
4. Edge latency stays inside the agreed p95 targets across repeated samples.
5. All application pods meet the baseline security posture.
6. Staging exposure matches the approved public surface.
7. Drift between source, image, and running bundle is automatically detected.

## Suggested follow-up artifacts

After this plan starts execution, add:

1. a service scorecard document under `docs/operations`,
2. a postmortem for the staging gateway drift incident,
3. a short decision record if AI worker model or staging capacity strategy changes materially.