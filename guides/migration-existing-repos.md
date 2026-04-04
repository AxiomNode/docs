# Migration Of Existing Repositories To Gateway And BFF

## Goal

Adapt existing repositories to the target architecture without interrupting current runtime behavior.

## Migrated repositories

- microservice-quizz
- microservice-wordpass
- microservice-users
- ai-engine

## Incremental strategy

1. Preserve compatibility of current microservice endpoints during migration.
2. Define and version internal contracts in `contracts-and-schemas`.
3. Consume domain services through BFFs and expose only gateway routes externally.
4. Gradually remove direct external access to domain microservices.

## Recommended operational changes

- Run domain microservices in private cluster networking.
- Allow external ingress only through `api-gateway`.
- Keep private docs enabled for internal support/debugging.
- Consolidate edge authentication in gateway and contextual auth in BFFs.

## Migration completion criteria

- Mobile and backoffice consume gateway routes only.
- No external clients call domain microservices directly.
- Versioned contracts are published for each service.
