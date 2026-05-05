# Architecture Document Template

Last updated: 2026-05-03.

## Purpose

Use this template for architecture documents that explain stable structure, boundaries, and design consequences across one or more repositories.

## Template

```md
# <Architecture topic>

Last updated: YYYY-MM-DD.

## Scope

State what part of the platform this architecture document covers.

## Why this matters

- Describe the problem this structure solves.
- Clarify why this document exists instead of a repository-local note.

## Context and boundaries

### In scope

- <system, repository, bounded context>

### Out of scope

- <adjacent area not owned here>

## Structural view

- Describe the main building blocks.
- Explain how they are connected.
- Link to diagrams or C4 views when they already exist.

## Key flows

1. Describe the most important runtime or design flows.
2. Keep the flow architectural rather than operational.

## Design constraints

- Contracts, security, latency, ownership, deployment, or compliance constraints.

## Tradeoffs and consequences

- What this design makes easier.
- What this design makes harder.
- What future changes must preserve.

## References

- Link to related ADRs, capability dossiers, and operations documents.
```

## Usage notes

- Prefer this template when the document explains stable structure rather than operator procedure.
- Avoid turning architecture docs into runbooks or implementation checklists.