# Deployment And Rollout Automation

Last updated: 2026-05-03.

## Purpose

This capability packages runtime services, publishes images, and rolls environments forward through the centralized infrastructure pipeline in `platform-infra`.

## Functional objective

- build and publish deployable artifacts consistently
- roll staging automatically for covered repositories
- keep rollout policy, manifests, and overlays centralized in one repository

## Ownership and boundaries

### Main owners

- `platform-infra`

### Upstream actors

- covered runtime repositories dispatching builds on `main`

### Explicit boundaries

- owns centralized packaging and rollout policy through GitHub Actions workflows and Kubernetes manifests
- delegates repository-specific validation to service repos before dispatch
- does not own business correctness of runtime services beyond build, deploy, and smoke outcomes

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| Workflow | service-repo dispatch jobs | upstream trigger into infra pipeline |
| Workflow | `.github/workflows/build-push.yaml` | image publication path |
| Workflow | deploy workflows in `platform-infra` | staging rollout path |
| Artifact | `kubernetes/base/` and `kubernetes/overlays/*` | desired cluster state |

### Inputs

- dispatch events from service repositories
- image tags, source refs, and workflow inputs
- manifests, overlays, and secret-backed environment configuration

### Outputs

- published GHCR images
- rollout attempts and deployment receipts
- updated staging or target cluster state

### Contract sources

- workflow inputs across service repos and `platform-infra`
- manifest and overlay conventions in `kubernetes/base/` and `kubernetes/overlays/`
- secret dependency matrix in `secrets/config/required-repos.json`

## Main functional flow

1. Service repo CI succeeds on `main`.
2. The service repo dispatches the infra build workflow.
3. `platform-infra` builds and publishes a GHCR image for the requested service.
4. Deploy workflows apply Kustomize overlays and push the cluster toward the desired state.
5. Smoke checks, health probes, and workflow logs determine whether the rollout converged.

## Persistence and runtime state

### Owned persistence

- workflow history and release metadata
- Kubernetes manifests and overlay state stored in Git

### Runtime state

- deployed cluster state after rollout
- rollout behavior depends on current image, secret, and cluster conditions
- the persisted llama target override in `ai-engine-api` can change effective behavior after deploy

### State risks

- manifests can diverge from effective runtime if routing-state holders override targets after deployment
- partial rollout or failed deploy can leave staging in an intermediate state
- older images may remain live while workflows have already advanced

## Observability evidence

### Health signals

- deployment smoke checks and service health probes

### Operational signals

- GitHub Actions build and deploy runs
- rollout logs and cluster events
- packaged backoffice version surfaced through operational diagnostics

### Verification evidence

- service-repo CI before dispatch
- post-build publication receipts
- post-deploy health validation and dashboard checks

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Build failure | broken source, dependency, or packaging step | no image publication | failed build run | fix source or packaging and rerun |
| Deploy failure | bad manifest, unhealthy image, cluster issue | target environment does not converge | deploy logs, skipped smoke tests | rollback deployment or correct manifests |
| Effective-runtime mismatch | runtime override still points elsewhere | deployed image is healthy but traffic still uses an old or temporary target | routing inspection and smoke checks | clear or reapply runtime overrides after deploy |

## Security and access model

- workflow dispatch and registry access depend on scoped tokens and secrets
- deployment automation is high privilege and must be tightly governed
- build and deploy logs may expose sensitive topology or secret names if not curated carefully

## Deployment and rollback implications

### Deployment impact

- this capability is itself the deployment plane, so rollout ordering and workflow gating matter directly
- covered repos must pass their own CI before entering the centralized chain

### Rollback impact

- rollback usually happens through a new infra action or manual cluster correction
- service rollbacks must consider image tag, manifest compatibility, and runtime-state holders such as routing overrides

## Related capabilities

- [Secrets governance](./secrets-governance.md)
- [Runtime routing control](../operations/runtime-routing-control.md)
- [Public edge routing](../edge/public-edge-routing.md)
- [Monitoring and alerting](./monitoring-and-alerting.md)

## Open questions

- which deploy failures are frequent enough to deserve more explicit runbooks
- whether every covered service has equally strong post-deploy smoke coverage

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Platform functional overview](../../foundations/platform-functional-overview.md)
- [Primary validation workflows](../../validation/primary-workflows-validation.md)
