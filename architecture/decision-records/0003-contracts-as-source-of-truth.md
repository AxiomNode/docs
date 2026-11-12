# ADR 0003 - Contracts repository as source of truth

- Status: Accepted
- Date: 2026-04-22

## Context

Multiple services and channels exchange request/response payloads. Without a single source of truth, schemas drift and consumers ship incompatible code.

## Decision

`contracts-and-schemas` is the canonical source for OpenAPI specs (per-service), JSON Schemas (gameplay objects, categories, queries), and event schemas. `shared-sdk-client` regenerates clients (TypeScript, Python, Kotlin) from those contracts. Services consume only the SDK; they do not redefine schemas locally.

CI enforces the chain: contracts validated -> SDK regenerated -> consumers compiled against new SDK.

> **Status note (2026-11-12): partially implemented.** Contracts are validated in CI, but nothing regenerates the SDK: `shared-sdk-client` is a hand-written internal TypeScript library and there are no Python or Kotlin clients. The OpenAPI files cover the internal services only (not the gateway or the BFFs), and the event schemas have no producer or consumer. Treat the chain above as the intended direction, not as the current behaviour.

## Alternatives considered

1. Per-service schemas duplicated locally: rejected; produced silent drift historically.
2. Generation from service code (annotations) into a contracts publisher: rejected for now; contracts are intentionally hand-curated to remain implementation-independent.

## Consequences

- Breaking contract changes block downstream services until updated.
- Contributors must understand the contract->SDK->consumer flow.
- Validation tooling lives in `contracts-and-schemas/scripts/validate_contracts.py` and is a CI gate.

## Related

- [QA-M1 in quality attributes](../quality-attributes.md)
- [shared-sdk-client adoption](../../guides/governance/repository-standards-adoption.md)
