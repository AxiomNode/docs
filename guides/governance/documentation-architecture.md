# Documentation Architecture

Last updated: 2026-05-03.

## Purpose

This guide defines how AxiomNode documentation is structured, which document types exist, and where each kind of knowledge should live.

The goal is to keep the documentation set useful for real engineering work: current-state accurate, easy to navigate, and explicit about ownership.

## Documentation layers

### Cross-repository architecture

Store in `docs/architecture/`.

Use for:

- platform topology and repository map
- domain concepts and system boundaries
- architectural decisions and stable sequence views
- quality attributes and glossary material

Avoid here:

- operator checklists
- environment-specific rollout steps
- repository-local implementation detail

### Cross-repository guides

Store in `docs/guides/`.

Use for:

- capability dossiers
- engineering guidance that helps apply the architecture
- cross-repository communication rules, validation rules, and adoption guidance
- templates used to keep documents consistent

Guide subfolders should remain purpose-driven:

- `foundations/`: conceptual entry points and platform-wide framing
- `communication/`: service and edge interaction rules
- `experience/`: UI and channel-specific engineering guidance
- `governance/`: documentation, repository, and delivery governance
- `validation/`: test, smoke, and verification workflows
- `capabilities/`: implementation-backed functional dossiers
- `templates/`: reusable document skeletons

### Cross-repository operations

Store in `docs/operations/`.

Use for:

- deployment and rollback policy
- environment and secret handling
- incident response and recovery runbooks
- CI/CD governance and hardening rules

Avoid here:

- long-form architectural explanation better suited to `architecture/`
- repository-local setup notes that belong beside the code

### Repository-local documentation

Store in each owning repository.

Use for:

- repository responsibility and local runtime role
- local development workflow and commands
- owned routes, storage, tests, and release constraints
- local architecture notes that do not need platform-wide duplication

## Document types

### Architecture document

Answers: what exists, why it is shaped this way, and how major parts relate.

Preferred sections:

- scope
- context and boundaries
- structural view
- key flows or decisions
- constraints and consequences

### Guide

Answers: how to understand or apply a platform concern consistently.

Preferred sections:

- purpose
- audience or scope
- decision rules
- examples or references

### Capability dossier

Answers: how one capability really behaves across code, runtime state, failure modes, and tests.

Preferred sections:

- ownership and boundaries
- entrypoints and contracts
- runtime state and risks
- observability and rollback implications

### Operations runbook or policy

Answers: what to do, when to do it, how to validate it, and how to recover.

Preferred sections:

- trigger or preconditions
- execution steps
- validation
- rollback or escalation

### Repository README

Answers: what this repository owns, how to run it, and how it fits the platform.

Preferred sections:

- responsibility and boundaries
- local setup and commands
- runtime surface
- dependencies and deployment notes

## Placement rules

Use this decision sequence before adding a new document:

1. If the content is owned by one repository only, place it in that repository first.
2. If it spans multiple repositories and explains stable structure, place it in `architecture/`.
3. If it spans multiple repositories and explains execution or recovery, place it in `operations/`.
4. If it spans multiple repositories and explains how to apply a rule or analyze a capability, place it in `guides/`.
5. If it is only reusable scaffolding, place it in `guides/templates/`.

## Normalization rules

- keep documents in English
- prefer one clear purpose per file
- avoid mixing current-state documentation with roadmap, retrospective-only, or audit-history content
- cross-link related documents instead of duplicating long summaries
- distinguish static deployment state from effective runtime state whenever overrides exist
- include last-updated metadata on maintained guides, architecture documents, and runbooks

## Recommended minimum set for a useful project

For a project or repository to be usefully documented, it should have at least:

1. One repository README explaining responsibility, setup, runtime surface, and deployment notes.
2. One architecture entry when the project changes platform structure or introduces a new bounded context.
3. One operations document when the project introduces rollout, secret, recovery, or production-only behavior.
4. One capability dossier when the project exposes behavior that crosses repositories or runtime state holders.

## Related templates

- [Capability analysis template](../templates/capability-analysis-template.md)
- [Architecture document template](../templates/architecture-document-template.md)
- [Operations runbook template](../templates/operations-runbook-template.md)
- [Repository documentation template](../templates/repository-documentation-template.md)