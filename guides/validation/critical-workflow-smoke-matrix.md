# Critical Workflow Smoke Matrix

Last updated: 2026-05-03.

## Purpose

Map each primary cross-repository workflow to the best currently available automated smoke or end-to-end evidence, and make the remaining automation gaps explicit.

This document complements [Primary validation workflows](./primary-workflows-validation.md):

- that guide defines what each workflow must prove,
- this matrix shows what already exists to prove it automatically,
- and where manual evidence is still required.

## Reading rules

- `Automated`: there is at least one existing executable smoke or E2E check that exercises the workflow path directly.
- `Partial`: there is automated evidence for part of the path, but not enough to prove the whole workflow end to end.
- `Manual`: the current proof still depends on operator or tester execution.

## Workflow matrix

| Workflow | Intent | Best current automated evidence | Current mode | Main gaps to close next |
|---|---|---|---|---|
| `WF-01` Mobile profile read/update | prove mobile identity and profile persistence through the edge path | `platform-infra/environments/dev/scripts/smoke-edge.sh` and `secrets/scripts/run-nightly-edge-smoke.sh` now execute unauthorized `GET`, invalid `PUT`, valid `GET`, valid `PUT`, and a verifying re-read for `/v1/mobile/player/profile`; repository tests still cover `microservice-users/src/tests/users.integration.test.ts`, `firebaseAuthService.test.ts`, and `bff-mobile/src/tests/mobile.test.ts` | Automated | Intentionally does not pin optional upstream identity fields such as `email` and `photoUrl`, because those originate in `microservice-users` identity resolution and would make the edge smoke more brittle than useful. |
| `WF-02` Mobile game discovery (random) | prove curated quiz and word-pass discovery through the public edge | `platform-infra/environments/dev/scripts/smoke-edge.sh`, `secrets/scripts/run-nightly-edge-smoke.sh`, `platform-infra/scripts/smoke-stg-edge.sh` call the mobile random routes through `api-gateway` | Automated | The existing smokes confirm route health and auth, but do not assert payload shape deeply enough for lobby-contract regression detection. |
| `WF-03` Mobile game start from inventory-backed generate routes | prove mobile generate routes remain inventory-backed and do not fall through to runtime AI generation | `platform-infra/environments/dev/scripts/smoke-edge.sh` and `secrets/scripts/run-nightly-edge-smoke.sh` now reject invalid generate payloads with `400` and require valid generate responses to return `{ gameType, generated }` for both quiz and word-pass; repository tests still prove that `bff-mobile` calls `/games/models/random` in `bff-mobile/src/tests/mobile.test.ts` | Partial | The smoke now discriminates route validation and playable envelope shape, but it still does not prove from runtime logs or upstream traces that the request stayed on `/games/models/random` without any hidden live-generation fallback. |
| `WF-04` Offline play and deferred event sync | prove continuity offline and reconciliation on reconnect | `platform-infra/environments/dev/scripts/smoke-edge.sh` and `secrets/scripts/run-nightly-edge-smoke.sh` now post a deferred event batch to `/v1/mobile/games/events`, verify converged profile stats, and replay the same batch to confirm deduplication; mobile unit tests still cover offline cache behavior in `mobile-app/App/composeApp/src/commonTest/kotlin/es/sebas1705/axiomnode/data/repositories/GamesRepositoryOfflineFallbackTest.kt` and `GamesRepositoryCatalogCacheTest.kt` | Automated | The smoke now proves reconciliation and deduplication through the edge path, but it still does not exercise the actual client-side offline banner, local queue persistence, or reconnect trigger UX. |
| `WF-05` AI content production for catalog refresh | prove AI-generated content enters quiz/word-pass inventory and becomes selectable by mobile-facing routes | `platform-infra/environments/dev/scripts/smoke-edge.sh` and `secrets/scripts/run-nightly-edge-smoke.sh` now require `ai-engine-api` health, execute both `microservice-quizz:/games/generate` and `microservice-quizz:/games/generate/process/wait`, verify `task.created >= 1`, confirm `GET /games/history` grows by that amount, and re-check selectable inventory via `GET /games/models/random`; `platform-infra/environments/dev/scripts/smoke-wf05-quiz.sh` now narrows `random` to the exact process window and requires the returned item id to belong to freshly persisted quiz rows; `platform-infra/environments/dev/scripts/smoke-wf05-wordpass.sh` offers the equivalent word-pass chain when `microservice-wordpass` is started with `BATCH_GENERATION_ENABLED=false`; `ai-engine/src/scripts/install/smoke_test.sh` and repository tests still cover segmented AI auth, catalog refresh, and generation internals | Partial | The narrow smokes now prove direct quiz and word-pass AI generation, observable persistence, and exact random traceability, but the full edge smoke still does not cover word-pass refresh and the workflow still does not replay the refreshed inventory through a full mobile chain in the same run. |
| `WF-06` Runtime routing diagnostics and rollback | prove operator override, smoke use of the affected route, and rollback to defaults | `platform-infra/environments/dev/scripts/smoke-edge.sh` and `secrets/scripts/run-nightly-edge-smoke.sh` now cover both variants: service-target override via `/v1/backoffice/service-targets/:service` plus affected catalogs request, and dedicated AI-target override via `/v1/backoffice/ai-engine/target` with readback and reset to `env`; repository tests still cover local state holders in `api-gateway`, `bff-backoffice`, and `ai-engine` | Automated | The smoke now covers both operator-facing override families, but it still does not detect split-brain mismatches across BFF, gateway, and ai-engine holders after partial sync failures. |

## Existing automation surfaces

### Edge and staging smoke

- `platform-infra/environments/dev/scripts/smoke-edge.sh`
- `secrets/scripts/run-nightly-edge-smoke.sh`
- `platform-infra/scripts/smoke-stg-edge.sh`

These scripts are the strongest current cross-repository smoke assets. They already validate public-edge reachability, auth gating, mobile profile round-trip, selected mobile routes, selected backoffice routes, and staging rollouts.

### Operator E2E smoke

- `backoffice/src/e2e/smoke.spec.ts`

This suite validates backoffice boot integrity and login-gate usability, but it does not yet prove deeper operational workflows such as routing override, diagnostics drill-down, or AI incident recovery.

### Service and client tests that reduce smoke blind spots

- `api-gateway/src/tests/proxy.test.ts`
- `bff-mobile/src/tests/mobile.test.ts`
- `bff-backoffice/src/tests/backoffice.test.ts`
- `microservice-users/src/tests/users.integration.test.ts`
- `mobile-app/App/composeApp/src/commonTest/kotlin/es/sebas1705/axiomnode/data/network/AuthHttpClientRoutesTest.kt`
- `mobile-app/App/composeApp/src/commonTest/kotlin/es/sebas1705/axiomnode/data/network/GamesHttpClientRoutesTest.kt`
- `mobile-app/App/composeApp/src/commonTest/kotlin/es/sebas1705/axiomnode/data/repositories/GamesRepositoryOfflineFallbackTest.kt`

These tests do not replace a cross-repository smoke, but they narrow the uncertainty around route shape, auth headers, offline behavior, and local orchestration.

## Recommended execution order

1. Run repository-local tests for the slices you touched.
2. Run `platform-infra/environments/dev/scripts/smoke-edge.sh` for local edge integration changes.
3. Run `backoffice/src/e2e/smoke.spec.ts` when the operator shell or login surface changes.
4. Run `platform-infra/scripts/smoke-stg-edge.sh` after staging rollout when the affected capability is already deployed.
5. Attach manual evidence only for the rows still marked `Manual` or `Partial`.

## Priority automation backlog derived from this matrix

1. Add one reconnect-oriented executable flow for `WF-04` that proves event upload and stats convergence.
2. Tighten `WF-03` and `WF-05` assertions so they distinguish healthy inventory-backed generation from degraded or AI-routed fallback paths.
3. If routing incidents continue, add a stronger mismatch-detection check that compares BFF, gateway, and ai-engine readback instead of only verifying the operator-facing success path.
4. Keep `WF-01` focused on edge-owned profile semantics and avoid coupling it to optional upstream identity fields unless a regression history justifies that tighter binding.

## Related documents

- [Primary validation workflows](./primary-workflows-validation.md)
- [Functionality review baseline](../../operations/functionality-review-baseline.md)
- [Local edge integration](../communication/local-edge-integration.md)