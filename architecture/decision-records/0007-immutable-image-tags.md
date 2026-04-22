# ADR 0007 - Immutable image tags for staging and production

- Status: Accepted
- Date: 2026-04-22

## Context

Mutable tags such as `:latest` make rollouts non-deterministic and hide what is actually running. Auditing past incidents required image SHAs, not floating tags.

## Decision

- Every build publishes an immutable `:sha` tag derived from the commit.
- Floating environment tags (`:stg`, `:prod`) are also published but only as pointers; deployments may reference either depending on overlay policy.
- `:latest` is forbidden in any manifest; CI guard in `validate-infra` fails the pipeline if found.
- Production publication of `:prod` requires a manual `publish_prod_tag` dispatch input.

## Alternatives considered

1. Always-latest with image pull policy `Always`: rejected; non-auditable rollouts.
2. SHA-only without floating tags: rejected; floating tags are useful for overlay readability and recovery.

## Consequences

- Predictable rollouts and traceable incident analysis.
- Emergency recovery may still use floating tags; documented in runbook.
- Operators must understand the difference between declared tag and resolved digest.

## Related

- [Deployment strategy](../../operations/deployment-strategy.md)
- [QA-S3 in quality attributes](../quality-attributes.md)
