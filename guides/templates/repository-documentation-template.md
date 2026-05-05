# Repository Documentation Template

Last updated: 2026-05-03.

## Purpose

Use this template to normalize repository-local README files or repository-local documentation landing pages.

## Template

```md
# <Repository name>

Last updated: YYYY-MM-DD.

## Responsibility

State what this repository owns and what it explicitly does not own.

## Runtime role

- Main service, library, workflow, or UI role
- Upstream and downstream dependencies

## Local setup

### Prerequisites

- Languages, runtimes, package managers, or tools

### Main commands

- install
- run
- test
- lint or validation

## Runtime surface

- Routes, jobs, screens, schemas, or exported modules owned here
- Persistent state owned here

## Dependencies and contracts

- Cross-repository dependencies
- Schema or SDK dependencies
- Secrets or environment dependencies

## Deployment and operations notes

- Build and deployment path
- Smoke checks
- Rollback constraints

## References

- Link to central docs and local deeper docs.
```

## Usage notes

- Keep the README focused on the owning repository rather than repeating platform-wide narrative.
- Link back to central cross-repository documents instead of copying them.