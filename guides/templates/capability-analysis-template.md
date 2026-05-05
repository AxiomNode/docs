# Capability Analysis Template

Last updated: 2026-05-03.

## Purpose

Use this template to document one AxiomNode capability in a repeatable way.

The goal is to keep future functional studies comparable across player-facing, operator-facing, AI, and platform capabilities.

## Recommended filename

Use a descriptive guide name such as:

- `player-identity-capability.md`
- `ai-generation-capability.md`
- `deployment-rollout-capability.md`

## Template

```md
# <Capability name>

Last updated: YYYY-MM-DD.

## Purpose

State in 2-4 lines what problem this capability solves and why it exists.

## Functional objective

- Describe the user, operator, or platform outcome this capability must achieve.
- Distinguish primary objective from secondary or supporting behavior.

## Ownership and boundaries

### Main owners

- `<repository or service>`
- `<repository or service>`

### Upstream actors

- `<client, operator, service, workflow>`

### Downstream dependencies

- `<service, datastore, external provider, workflow>`

### Explicit boundaries

- What this capability owns directly.
- What it delegates to another repository or service.
- What is intentionally out of scope.

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| HTTP route | `<method> <path>` | `<purpose>` |
| UI flow | `<screen or action>` | `<purpose>` |
| Job/workflow | `<workflow or script>` | `<purpose>` |

### Inputs

- Request/query/body fields
- headers/tokens/secrets
- files/events/scheduled triggers

### Outputs

- response payloads
- persisted side effects
- downstream calls
- emitted artifacts or workflow results

### Contract sources

- `contracts-and-schemas/...`
- `shared-sdk-client/...`
- repo-local schemas or validators

## Main functional flow

1. Describe the normal end-to-end path in 4-8 ordered steps.
2. Keep the flow concrete enough to show decision points.
3. Mention where validation, persistence, or forwarding happens.

## Persistence and runtime state

### Owned persistence

- database tables
- files
- cached values
- generated artifacts

### Runtime state

- environment defaults
- persisted overrides
- in-memory state
- effective target resolution if applicable

### State risks

- stale state after restart
- drift between persisted and configured values
- concurrency or overwrite risk

## Observability evidence

### Health signals

- health endpoints
- readiness/liveness conditions

### Operational signals

- logs
- metrics
- traces
- dashboards
- alerts

### Verification evidence

- tests
- smoke checks
- workflow checks
- manual validation paths

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| `<failure>` | `<cause>` | `<impact>` | `<signal>` | `<response>` |

## Security and access model

- Authentication requirements
- Authorization/role gating
- Secrets involved
- Sensitive data handled

## Deployment and rollback implications

### Deployment impact

- Which repositories must change together.
- Whether contracts or SDKs must be regenerated.
- Whether rollout ordering matters.

### Rollback impact

- What happens if only one repo is rolled back.
- Whether persisted state must also be reverted or migrated.
- Any known safe rollback boundary.

## Open questions

- Unknowns that still need confirmation in code or runtime.
- Ambiguities between documented and effective behavior.

## References

- Link to the relevant row in `platform-capability-matrix.md`.
- Link to the owning repository README or detailed docs.
- Link to key architecture or operations documents.
```

## Usage notes

- Prefer one capability per file.
- Keep repository facts concrete and current-state only.
- If a capability spans multiple repositories, document the control boundary explicitly.
- Distinguish static configuration from effective runtime behavior whenever overrides exist.
- When a capability is operationally critical, fill the failure table before adding extra narrative.
