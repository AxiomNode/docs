# Repository Standards Adoption Guide

Last updated: 2026-04-22.

## Goal

This guide explains how to adopt the repository minimum standards in an existing AxiomNode repository without blocking delivery unnecessarily.

Use this guide when:

- bringing an older repository up to the current engineering baseline
- creating a new repository that must align with workspace governance
- auditing a repository that only partially satisfies the standards in [Repository minimum standards](../operations/repository-minimum-standards.md)

## Adoption order

Apply the standards in the following order.

### 1. Classify the repository correctly

Decide whether the repository is primarily:

- an executable runtime repository
- a platform or shared-code repository
- a documentation-only repository

This classification determines whether runtime deployment and strict coverage gates apply directly.

### 2. Document the repository in English

Before tightening gates, make the repository understandable.

Minimum actions:

- update the root README in English
- document repository responsibility and boundaries
- document local development workflow
- document CI expectations and release behavior

### 3. Make architecture boundaries explicit

If the repository is executable, identify and reduce unnecessary coupling between:

- transport and HTTP handlers
- application orchestration
- domain logic
- persistence or infrastructure glue

The first target is not perfection. The first target is a reviewable structure with visible dependency boundaries.

### 4. Make CI mandatory before increasing deployment automation

Every repository should have a required validation workflow before any further automation is treated as trustworthy.

Minimum expected CI content for executable repositories:

- dependency installation
- build or compile validation when relevant
- automated tests
- mandatory coverage verification when the repository already supports it reliably

### 5. Raise coverage to the enforced floor

For executable repositories, the target is an enforced 90% threshold on the richest reliable metric set the active harness can support.

Recommended rollout:

1. stabilize failing tests first
2. add tests to high-impact uncovered modules
3. enable the threshold in CI only after the repository can pass it consistently
4. treat regressions below the threshold as blocking failures

### 6. Connect deployable runtime repositories to automatic staging

Only after CI and repository quality gates are trustworthy should a repository be considered for the covered automatic `stg` deployment chain.

Admission questions:

1. does the repository publish a runtime image?
2. is it part of the centralized `platform-infra` packaging and rollout path?
3. is automatic staging rollout operationally safe for that repository?

If the answer is no, document the reason explicitly instead of implying that the repository is already part of the automatic chain.

## Adoption profiles by repository class

### Executable runtime repository

Target end state:

- explicit architecture boundaries
- English documentation
- required CI validation
- mandatory tests
- enforced coverage threshold at or above 90%
- automatic continuation to `stg` when the repository belongs to the covered runtime set

### Platform or shared-code repository

Target end state:

- English documentation
- explicit ownership boundaries
- required validation workflows
- executable tests and coverage enforcement where the repository exposes a real testable code surface
- explicit documentation when runtime deployment is not applicable

### Documentation-only repository

Target end state:

- English documentation
- required documentation validation workflow
- current-state accuracy
- clear ownership and cross-links

## Common migration traps

- enabling automatic deployment before repository CI is genuinely blocking
- claiming 90% coverage as policy without actually enforcing it in CI
- treating documentation language as optional and ending with mixed-language operational docs
- forcing all repositories into the same runtime-delivery model even when some are libraries, infra, or docs only
- tightening architecture language without documenting what the local boundary model actually is

## Suggested repository adoption checklist

- [ ] Repository class is explicit
- [ ] README is in English and describes ownership and workflow
- [ ] Architecture boundaries are visible and documented
- [ ] Required CI workflow exists
- [ ] Build and tests are blocking
- [ ] Coverage threshold is enforced where applicable
- [ ] Automatic staging continuation is documented as enabled or intentionally out of scope
- [ ] PR template is aligned with the repository class

## Related documents

- [Repository minimum standards](../operations/repository-minimum-standards.md)
- [Test coverage policy](../operations/test-coverage-policy.md)
- [Deployment strategy](../operations/deployment-strategy.md)
- [CI/CD workflow map](../operations/cicd-workflow-map.md)