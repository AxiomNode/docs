# Primary Validation Workflows

Last updated: 2026-04-25.

## Purpose

Provide a practical validation script for the most important AxiomNode runtime workflows, with explicit expected outcomes and evidence to collect.

Use [Critical workflow smoke matrix](./critical-workflow-smoke-matrix.md) alongside this guide to see which workflows already have executable smoke or E2E coverage and which ones still require manual evidence.

When you need a narrow local repro for `WF-05` without bringing up gateway, BFFs, or backoffice, use [WF-05 quiz local smoke](./wf05-quiz-local-smoke.md) or [WF-05 word-pass local smoke](./wf05-wordpass-local-smoke.md).

## Scope

This guide focuses on cross-repository runtime flows:

- mobile player profile lifecycle
- game discovery and game start from curated model inventory
- offline event buffering and sync
- AI content production for microservice inventories
- operator diagnostics for runtime routing

## Validation baseline

Use this baseline before running the workflows:

1. `api-gateway`, `bff-mobile`, `microservice-quizz`, and `microservice-wordpass` are healthy.
2. `ai-engine` health endpoint is reachable for ingestion and diagnostics flows.
3. At least one category exists for quiz and wordpass in catalogs.
4. Test user can authenticate and obtains a valid bearer token.
5. Correlation id logging is enabled in gateway and BFF services.

## WF-01 Mobile profile read/update

### Goal

Validate that mobile identity and profile persistence work end to end through the edge path.

### Steps

1. Call `GET /v1/mobile/player/profile` with a valid bearer token.
2. Confirm response `200` with `profile.playerId` and `stats` payload.
3. Call `PUT /v1/mobile/player/profile` with `preferredLanguage` and optional profile fields.
4. Repeat `GET /v1/mobile/player/profile` and verify updated fields are persisted.

### Expected result

- Read and update requests return `200`.
- Player identity remains stable across calls.
- Profile data and stats snapshot are returned in a consistent shape.

### Negative checks

- Missing or invalid identity must return `401`.
- Invalid payload must return `400` with validation details.

### Evidence

- API responses for initial read, update, and final read.
- Gateway and BFF logs with the same correlation id.

## WF-02 Mobile game discovery (random)

### Goal

Validate quiz and wordpass discovery from curated model inventory.

### Steps

1. Call `GET /v1/mobile/games/quiz/random` with language and category query parameters.
2. Call `GET /v1/mobile/games/wordpass/random` with equivalent filters.
3. Validate payload schema expected by mobile game lobby screens.

### Expected result

- Both endpoints return `200`.
- Returned game payloads are playable without additional generation calls.
- Query validation happens at gateway/BFF boundaries.

### Negative checks

- Invalid query ranges (for example invalid difficulty) must return `400`.

### Evidence

- Request and response samples for quiz and wordpass random endpoints.
- Logs confirming forwarding to `/games/models/random` in each microservice.

## WF-03 Mobile game start from inventory-backed generate routes

### Goal

Validate that mobile generate routes build a playable session from microservice model inventory and do not execute runtime AI generation.

### Steps

1. Call `POST /v1/mobile/games/quiz/generate` with `categoryId`, `language`, and optional difficulty.
2. Call `POST /v1/mobile/games/wordpass/generate` with required fields.
3. Verify in logs that BFF calls microservices using `GET /games/models/random`.
4. Confirm response includes `gameType` and `generated` object for immediate play.

### Expected result

- Generate endpoints return `200` with a playable payload.
- Upstream call path is inventory lookup only (`/games/models/random`).
- No direct runtime call to AI generation pipeline is triggered by these mobile routes.

### Negative checks

- Missing required generate payload fields must return `400`.
- Empty upstream inventory should return `502` with a clear message.

### Evidence

- BFF logs for generated requests and upstream URL.
- Response payload snapshots proving generated content is sourced from inventory.

## WF-04 Offline play and deferred event sync

### Goal

Validate user continuity when connectivity drops and reliable reconciliation when connectivity returns.

### Steps

1. Disable client network and complete one or more rounds locally.
2. Confirm local history/stats update in mobile UI while offline.
3. Re-enable network and trigger manual or automatic sync.
4. Call `POST /v1/mobile/games/events` and confirm accepted event batch.
5. Read profile again and verify aggregated stats include synced rounds.

### Expected result

- Offline mode keeps gameplay and local progress available.
- Sync endpoint returns successful `synced` count on reconnect.
- Profile stats converge with uploaded events after reconciliation.

### Negative checks

- Sync attempts while offline should be blocked by client connectivity checks.
- Duplicate event uploads should not inflate stats.

### Evidence

- Mobile screenshots for offline banner/state.
- Sync request/response payload and resulting profile stats diff.

## WF-05 AI content production for catalog refresh

### Goal

Validate the operational flow where AI produces content for microservice inventories, separate from mobile runtime game start.

### Steps

1. Execute quiz or word-pass AI generation through the service APIs, preferably `POST /games/generate` for a single item or `POST /games/generate/process/wait` when a blocking batch result is needed.
2. Confirm the generation response returns `201` and includes `gameType` plus generated content or generated items.
3. Capture `GET /games/history` before and after a blocking generation run and verify persisted history grows by the reported `task.created` count.
4. Call `GET /games/models/random` on the same microservice and verify the selected item belongs to the exact persistence window created by that blocking generation run.
5. Re-run `WF-02` and `WF-03` to confirm mobile still consumes the refreshed inventory through the edge without route changes.

For narrow local repros, the dedicated scripts cover both branches independently:

- [WF-05 quiz local smoke](./wf05-quiz-local-smoke.md)
- [WF-05 word-pass local smoke](./wf05-wordpass-local-smoke.md)

### Expected result

- AI pipeline produces new inventory entries successfully.
- Persisted generation history grows in line with the reported created count.
- Microservices expose newly persisted entries through random model queries with traceable item identity.
- Mobile-facing flows consume refreshed inventory without route changes.

### Negative checks

- Failed ingestion should not corrupt existing inventory.
- Partial pipeline failures should be observable in diagnostics.

### Evidence

- Request and response samples for `POST /games/generate` or `POST /games/generate/process/wait`.
- Backoffice diagnostics snapshots before/after ingestion.
- Microservice history and random-query samples showing that a selected random item id belongs to the exact ids persisted by the validated generation window.

## WF-06 Runtime routing diagnostics and rollback

### Goal

Validate operator control of runtime targets and safe return to defaults.

### Steps

1. Open diagnostics in backoffice and read current effective targets.
2. Apply a valid temporary target override.
3. Run a smoke request that uses the affected route.
4. Revert to environment default target.
5. Confirm effective source changes from `override` back to `env`.

### Expected result

- Override and rollback both succeed with persisted state.
- Runtime behavior follows the active target source.
- Operator UI displays current source and target clearly.

### Negative checks

- Non-allowlisted hosts must be rejected.
- Unauthorized requests must not mutate routing state.

### Evidence

- Diagnostic panel screenshots for override and rollback states.
- Gateway/BFF logs for applied target and smoke request outcome.

## Exit criteria

A validation cycle is complete when:

1. All workflows `WF-01` to `WF-06` pass expected results.
2. Negative checks produce controlled and documented failures.
3. Evidence is attached to the validation report for each workflow.

## Related documents

- [Critical workflow smoke matrix](./critical-workflow-smoke-matrix.md)
- [WF-05 quiz local smoke](./wf05-quiz-local-smoke.md)
- [WF-05 word-pass local smoke](./wf05-wordpass-local-smoke.md)
- [Key sequence flows](../../architecture/key-sequence-flows.md)
- [Mobile app development guide](../experience/mobile-app-development-firebase.md)
- [Runtime routing and service targeting](../../operations/runtime-routing-and-service-targeting.md)