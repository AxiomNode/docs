# GitHub Governance Baseline

Last updated: 2026-04-22.

## Intent

This document defines the minimum GitHub governance baseline expected across AxiomNode repositories.

It complements the repository minimum standards, branch protection policy, PR templates, issue templates, and discussion templates by describing the target GitHub repository posture at a platform level.

## Baseline repository features

Every actively maintained AxiomNode repository should have the following GitHub features configured deliberately rather than by accident.

Minimum baseline:

- protected `main` branch
- blocking required checks aligned with the repository workflow map
- pull request template present
- issue templates present
- blank issues disabled when structured intake is required
- discussion templates prepared when the repository uses GitHub Discussions

## Recommended organization-level ruleset baseline

When GitHub organization rulesets are available, the following baseline should be applied as the default policy for repositories in scope.

### Core ruleset for `main`

Recommended defaults:

- target branch: `main`
- require pull requests before merge
- require at least one approving review
- dismiss stale approvals when the branch changes
- require conversation resolution before merge
- block force-pushes
- block branch deletion
- require the repository-specific validation checks documented in [Branch protection policy](./branch-protection-policy.md)

### Runtime-service overlay

For covered executable runtime repositories, additionally require that:

- the service CI workflow is a blocking check
- merges cannot proceed while test, coverage, lint, or compile failures remain inside that workflow
- staging continuation remains downstream of successful validation only

### Platform and docs overlay

For platform, shared-code, and documentation repositories:

- all correctness-preserving validation workflows must be blocking
- scheduled or observational workflows should not be configured as merge-blocking checks unless they are part of correctness validation

## Current discussion-category baseline

Where GitHub Discussions are enabled, repositories should use a small shared category set so intake stays predictable.

Recommended categories:

- `general`: cross-cutting discussion, design questions, or open technical context
- `ideas`: proposed improvements, architecture evolution, or workflow changes
- `q-a`: repository usage questions, local workflow questions, or operator questions

Operational note:

- discussion form files only become active when the repository has a matching discussion category slug configured in GitHub
- if Discussions are disabled, the template files are harmless but inactive
- poll categories should not be used for discussion forms because GitHub does not support forms for polls

## Intake-governance expectations

The combined expected posture for repository intake is:

- pull requests use the repository PR template
- issues use structured bug or change-request templates
- blank issues stay disabled unless a repository owner has a documented reason to allow free-form intake
- discussions, when enabled, use shared category forms for general, ideas, and Q&A threads

## Drift management

Repository owners should treat the following as governance drift:

- required workflows documented in docs but not configured in GitHub rulesets or branch protection
- blank issues re-enabled without an explicit reason
- discussions enabled with no template baseline or with inconsistent category naming
- repository-local community files that contradict the central governance docs

## Related documents

- [Repository minimum standards](./repository-minimum-standards.md)
- [Branch protection policy](./branch-protection-policy.md)
- [CI/CD workflow map](./cicd-workflow-map.md)
- [Repository standards adoption guide](../guides/repository-standards-adoption.md)