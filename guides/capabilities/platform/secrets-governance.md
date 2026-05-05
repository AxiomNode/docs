# Secrets Governance

Last updated: 2026-05-03.

## Purpose

This capability keeps sensitive configuration governed, synchronized, and auditable across repositories and environments through the dedicated `secrets` repository.

## Functional objective

- manage secret material centrally
- enforce placeholder hygiene and distribution rules
- support safe bootstrap and rotation workflows

## Ownership and boundaries

### Main owners

- `secrets`

### Main consumers

- all runtime and platform repositories

### Explicit boundaries

- owns centralized secret inventory, placeholder policy, and distribution rules
- delegates runtime use of those secrets to consumer repos and workflows
- does not replace repository-local config documentation, but constrains it and validates it

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| Source artifact | environment-specific secret material under `runtime/` | source of real secret values |
| Script | placeholder validation and audit scripts under `scripts/` | drift detection |
| Source artifact | repo dependency matrix in `config/required-repos.json` | environment/repo secret expectations |

### Inputs

- environment secret values
- placeholder definitions in consuming repos
- audit and bootstrap parameters

### Outputs

- validated placeholder state
- prepared secret bundles or runtime-ready files
- audit results and drift findings

### Contract sources

- `config/required-repos.json`
- scripts and runtime material in the `secrets` repository
- operational documentation under `secrets/docs/`

## Main functional flow

1. Secret values are organized per environment in the `secrets` repository.
2. Validation scripts compare consumer placeholders with the expected inventory and repo matrix.
3. Bootstrap or runtime-preparation steps distribute the right secret material into consumer workflows or local environments.
4. Infra and runtime repositories consume the prepared values during build and deployment.

## Persistence and runtime state

### Owned persistence

- secret inventories and environment-specific sources

### Runtime state

- distributed placeholder files across repositories
- actual runtime success depends on correct secret consumption outside this repo

### State risks

- placeholder drift
- inconsistent rotation state between environments
- consumer repo assumptions diverging from the canonical secret inventory

## Observability evidence

### Health signals

- successful secret preparation/bootstrap runs

### Operational signals

- audit outputs
- placeholder validation checks
- rotation or distribution status

### Verification evidence

- workflow checks in consuming repos
- local environment bootstrap success
- secret-audit outputs

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Placeholder drift | repo config changed without secret update | broken runtime or workflow startup | validation failures | realign placeholder and secret source |
| Secret exposure risk | bad handling or accidental commit | security incident | audit checks and review | rotate affected secrets and tighten process |
| Rotation mismatch | one environment updated while another lags | rollout works in one environment and fails in another | bootstrap and deployment failures | reconcile environment inventories and rerun |

## Security and access model

- this capability is itself security-critical and should be tightly access controlled
- secrets must never be published into generated docs or runtime logs
- emergency rotation procedures are part of the operational boundary

## Deployment and rollback implications

### Deployment impact

- rollout can fail even with healthy artifacts if secret distribution is wrong
- secret changes can force coordinated updates across infra and runtime repos

### Rollback impact

- rollback may require both application rollback and secret-state reconciliation
- rolling back code without restoring compatible secrets may not recover runtime behavior

## Related capabilities

- [Deployment and rollout automation](./deployment-and-rollout-automation.md)
- [Runtime routing control](../operations/runtime-routing-control.md)
- [Documentation and engineering guidance](./documentation-and-engineering-guidance.md)

## Open questions

- whether every consuming repo documents its secret ownership boundary clearly enough
- whether rotation cadence and audit evidence are equally strong across all environments

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
