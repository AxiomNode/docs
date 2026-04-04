# Deployment Strategy

## Environment model

- `dev`: fast feedback, lower-cost runtime profile, non-production data.
- `stg`: staging mirror for pre-release validation.
- `prod`: hardened runtime with stricter change controls.

## Current CI/CD behavior

1. Service repository CI runs build/test/lint checks.
2. Push to `main` in service repos dispatches `platform-infra` build workflow.
3. `platform-infra` builds and pushes images to GHCR.
4. `platform-infra` deploy workflow runs after successful build.
5. Automatic rollout is currently limited to `dev`.

## Promotion policy

- `stg` and `prod` are intentionally not auto-promoted in the current setup.
- Promotion to higher environments should be controlled by explicit release criteria.

## Versioning strategy

- Use SemVer for services and SDK packages.
- Maintain explicit contract versioning (`v1`, `v2`, ...).
- Define and communicate deprecation windows before removing contract support.
