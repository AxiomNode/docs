# Secrets Distribution Guide

## Source of truth

- The private repository `secrets` contains all real secrets.
- Runtime repositories must not commit real secrets.

## Copy model

1. Clone `secrets`.
2. Pick distribution: `dev`, `stg`, `prod`.
3. Run from `secrets`:

```bash
node scripts/prepare-runtime-secrets.mjs dev
```

4. This validates synchronization rules and copies only required file for each repository.

## Target path per repository

- `api-gateway/src/.env.secrets`
- `bff-mobile/src/.env.secrets`
- `bff-backoffice/src/.env.secrets`
- `microservice-quizz/src/.env.secrets`
- `microservice-wordpass/src/.env.secrets`
- `microservice-users/src/.env.secrets`
- `ai-engine/src/.env.secrets`

## Synchronization rule by distribution

For the same distribution (`dev`, `stg`, `prod`), shared keys must match across repositories that use them:

- `AI_ENGINE_API_KEY`
- `AI_ENGINE_GAMES_API_KEY`
- `AI_ENGINE_BRIDGE_API_KEY`
- `AI_ENGINE_STATS_API_KEY`
- `INTERNAL_API_TOKEN`

Between different distributions, values must be different.

## One-command flows

```bash
node scripts/prepare-runtime-secrets.mjs dev
node scripts/prepare-runtime-secrets.mjs stg
node scripts/prepare-runtime-secrets.mjs prod
node scripts/prepare-runtime-secrets.mjs all
```

If not all repositories are cloned locally, missing ones are skipped.

## Hardcoded secrets audit

```bash
node scripts/audit-hardcoded-secrets.mjs
```

This must be run from the private `secrets` repository before commits/releases.

## Centralized CI model

The private `secrets` repository can enforce validation for all repositories by:

1. Resolving required repositories per environment from `secrets/config/required-repos.json`.
2. Checking out each target repository in CI.
3. Running `audit-hardcoded-secrets.mjs --strict` from `secrets`.
4. Running `prepare-runtime-secrets.mjs all` to verify export flow.

## Compose behavior

All runtime compose files now read `.env.secrets` in addition to `.env` where applicable.
