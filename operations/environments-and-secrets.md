# Environments and Secrets

Last updated: 2026-04-05.

## Environment configuration model

- Non-sensitive defaults live in `.env` / `.env.example`.
- Real sensitive values are injected into `.env.secrets` from the private `secrets` repository.
- Shared key names are kept consistent within each distribution (`dev`, `stg`, `pro`).

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

## CI/CD-related tokens

- `PLATFORM_INFRA_DISPATCH_TOKEN` (service repositories): used to dispatch `platform-infra` build workflow on `main` pushes.
- `CROSS_REPO_READ_TOKEN` (`platform-infra`): used for cross-repo checkout during build matrix.
- `SECRETS_CROSS_REPO_READ_TOKEN` (`secrets`): used for centralized cross-repo secret audits.

## Service-level variable references

For detailed per-service variable lists, use each repository `README.md` and `.env.example` as the authoritative source.

## Known orphan variable

`SERVICE_COMMUNICATION_TOKEN` is still generated and validated by secrets tooling but is not currently consumed by runtime services.
