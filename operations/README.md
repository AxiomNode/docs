# Operations Index

Last updated: 2026-05-03.

This section contains current operational policy, rollout rules, environment guidance, and incident or recovery material for AxiomNode.

Use operations documents for execution-oriented content: what operators and maintainers do, in which order, under which constraints, and with which recovery path.

## Operational policy

- [Deployment strategy](./deployment-strategy.md)
- [CI/CD workflow map](./cicd-workflow-map.md)
- [Branch protection policy](./branch-protection-policy.md)
- [GitHub governance baseline](./github-governance-baseline.md)
- [Repository minimum standards](./repository-minimum-standards.md)
- [Functionality review baseline](./functionality-review-baseline.md)
- [Test coverage policy](./test-coverage-policy.md)
- [Coverage baseline report](./test-coverage-baseline.md)

## Runtime and recovery

- [Environments and secrets](./environments-and-secrets.md)
- [Secrets distribution guide](./secrets-distribution-guide.md)
- [Secrets rotation policy](./secrets-rotation-policy.md)
- [Runtime routing and service targeting](./runtime-routing-and-service-targeting.md)
- [AI services recovery runbook](./ai-services-recovery-runbook.md)
- [Threat model](./threat-model.md)
- [Retrospective process](./retrospective-process.md)

## When to write here

- when the document tells an operator or maintainer how to execute, validate, recover, or govern a workflow
- when environment differences, rollback constraints, or secret handling matter directly

## When not to write here

- do not put static architecture narratives here
- do not use this section for low-level repository setup that belongs in the owning repo README
- do not duplicate capability dossiers when the operational doc is only consuming their outputs