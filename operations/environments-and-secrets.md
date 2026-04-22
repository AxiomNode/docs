# Environments and Secrets

Last updated: 2026-04-05.

## Environment configuration model

- Non-sensitive defaults live in `.env` / `.env.example`.
- Real sensitive values are injected into `.env.secrets` from the private `secrets` repository.
- Shared key names are kept consistent within each distribution (`dev`, `stg`, `pro`).

## Ownership model

- `secrets` is the source of truth for real secret values and distribution policy.
- Each repository `README.md` and `.env.example` should describe concrete variable purpose and local usage.
- The central `docs` repository should describe environment model, token classes, and governance rules across repositories.

## Secret governance rules

- Rotate values through `secrets/scripts/bootstrap-secrets.mjs`.
- Never commit `.env.secrets` in runtime repositories.
- Validate consistency with:
	- `validate-secrets-sync.mjs`
	- `validate-no-placeholders.mjs`
- Audit hardcoded secrets with:
	- `audit-hardcoded-secrets.mjs`

## Shared runtime tokens (same value inside one distribution)

| Token | Main consumers |
|---|---|
| `API_TOKEN` | `microservice-quizz`, `microservice-wordpass`, `ai-engine` |
| `AI_ENGINE_API_KEY` | quiz/wordpass services, `bff-backoffice`, `ai-engine` |
| `AI_ENGINE_GAMES_API_KEY` | `ai-engine` |
| `AI_ENGINE_BRIDGE_API_KEY` | `bff-backoffice`, `ai-engine` |
| `AI_ENGINE_STATS_API_KEY` | `ai-engine` |
| `INTERNAL_API_TOKEN` | `api-gateway`, `bff-mobile`, `bff-backoffice` |
| `PRIVATE_DOCS_TOKEN` | quiz/wordpass/users services |

## Runtime-state-related variables to distinguish from secrets

Some operational behaviors depend on persisted runtime state rather than only on secret or environment variables.

Important examples:

- gateway ai-engine target override state
- backoffice service-target and preset state
- ai-engine llama target override state

These are not secret values, but they change effective runtime behavior and must be considered alongside environment configuration during incident analysis.

## CI/CD-related tokens

- `PLATFORM_INFRA_DISPATCH_TOKEN` (service repositories): used to dispatch `platform-infra` build workflow on `main` pushes.
- `CROSS_REPO_READ_TOKEN` (`platform-infra`): used for cross-repo checkout during build matrix.
- `SECRETS_CROSS_REPO_READ_TOKEN` (`secrets`): used for centralized cross-repo secret audits.

## Service-level variable references

For detailed per-service variable lists, use each repository `README.md` and `.env.example` as the authoritative source.

Those repository-local references should explain:

- what the variable controls locally
- whether it is a secret or a non-sensitive default
- whether a persisted runtime override can supersede it

## Known orphan variable

`SERVICE_COMMUNICATION_TOKEN` is still generated and validated by secrets tooling but is not currently consumed by runtime services.
