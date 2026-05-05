# WF-05 Word-pass Local Smoke

Last updated: 2026-05-03.

## Purpose

Run a narrow local smoke for `WF-05` without starting the full edge stack.

This smoke validates only the word-pass branch of the AI content production flow:

1. `ai-engine-api` is reachable.
2. `microservice-wordpass` can generate content through AI.
3. persisted word-pass history grows by the reported created count.
4. the resulting inventory is selectable through `/games/models/random`, and the returned item id belongs to the exact rows persisted by the blocking generation window.

## Scope

This is intentionally narrower than the full edge smoke.

It does not validate:

- quiz generation
- gateway or BFF routing
- mobile consumption after refresh
- backoffice diagnostics surfaces

## Prerequisites

1. Docker Desktop running.
2. Dev secrets injected for `ai-engine` and `microservice-wordpass`.
3. A local llama model already available for the selected `ai-engine` CPU or GPU profile.

## Minimal stack start

From the workspace root:

```bash
API_PORT=7001 docker compose -f ai-engine/src/docker-compose.yml --profile cpu up -d --build ai-cache llama-server-cpu ai-stats ai-api
docker compose -f microservice-wordpass/docker-compose.yml stop microservice-wordpass
docker compose -f microservice-wordpass/docker-compose.yml run -d --service-ports -e BATCH_GENERATION_ENABLED=false microservice-wordpass
```

The one-off `run` command is intentional: disabling the periodic batch scheduler removes local background generation that can keep `ai-engine` busy and make the smoke flaky.

## Run the smoke

From the workspace root:

```bash
bash platform-infra/environments/dev/scripts/smoke-wf05-wordpass.sh
```

## What it checks

- `GET http://localhost:7001/health`
- `GET http://localhost:7101/health`
- invalid `POST /games/generate` returns `400`
- valid `POST /games/generate` returns generated word-pass payload
- `POST /games/generate/process/wait` eventually reports `task.created >= 1`
- `GET /games/history` grows exactly by that created count
- `GET /games/models/random` returns a word-pass item whose id matches the exact persistence window created by `process/wait`

## Shutdown

```bash
docker compose -f microservice-wordpass/docker-compose.yml down --remove-orphans
docker compose -f ai-engine/src/docker-compose.yml --profile cpu down --remove-orphans
```

## Related documents

- [Primary validation workflows](./primary-workflows-validation.md)
- [Critical workflow smoke matrix](./critical-workflow-smoke-matrix.md)
- [Local edge integration](../communication/local-edge-integration.md)