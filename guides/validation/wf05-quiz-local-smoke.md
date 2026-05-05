# WF-05 Quiz Local Smoke

Last updated: 2026-05-03.

## Purpose

Run a narrow local smoke for `WF-05` without starting the full edge stack.

This smoke validates only the quiz branch of the AI content production flow:

1. `ai-engine-api` is reachable.
2. `microservice-quizz` can generate content through AI.
3. persisted quiz history grows by the reported created count.
4. the resulting inventory is selectable through `/games/models/random`, and the returned item id belongs to the exact rows persisted by the blocking generation window.

## Scope

This is intentionally narrower than the full edge smoke.

It does not validate:

- word-pass generation
- gateway or BFF routing
- mobile consumption after refresh
- backoffice diagnostics surfaces

## Prerequisites

1. Docker Desktop running.
2. Dev secrets injected for `ai-engine` and `microservice-quizz`.
3. A local llama model already available for the selected `ai-engine` CPU or GPU profile.

## Minimal stack start

From the workspace root:

```bash
docker compose -f ai-engine/src/docker-compose.yml --profile cpu up -d --build ai-cache llama-server-cpu ai-stats ai-api
docker compose -f microservice-quizz/docker-compose.yml up -d --build
```

## Run the smoke

From the workspace root:

```bash
bash platform-infra/environments/dev/scripts/smoke-wf05-quiz.sh
```

## What it checks

- `GET http://localhost:7001/health`
- `GET http://localhost:7100/health`
- invalid `POST /games/generate` returns `400`
- valid `POST /games/generate` returns generated quiz payload
- `POST /games/generate/process/wait` reports `task.created >= 1`
- `GET /games/history` grows exactly by that created count
- `GET /games/models/random` returns a quiz item whose id matches the exact persistence window created by `process/wait`

## Shutdown

```bash
docker compose -f microservice-quizz/docker-compose.yml down --remove-orphans
docker compose -f ai-engine/src/docker-compose.yml --profile cpu down --remove-orphans
```

## Related documents

- [Primary validation workflows](./primary-workflows-validation.md)
- [Critical workflow smoke matrix](./critical-workflow-smoke-matrix.md)
- [Local edge integration](../communication/local-edge-integration.md)