# Test Coverage Policy

## Scope

This document defines the minimum automated test-coverage rule for AxiomNode executable code repositories.

## Rule

All executable application scopes must maintain at least 90% coverage for the metrics their active test harness can report and gate reliably.

For Vitest-based repositories, the enforced minimum is 90% for:

- lines
- statements
- functions
- branches

For Python repositories using coverage.py, the enforced minimum is currently a 90% combined total coverage gate with branch instrumentation enabled.

The Python report also exposes a separate branch-coverage percentage for diagnostics, but the active CI failure threshold is evaluated on the combined total produced by coverage.py.

Function-level coverage is not emitted by the current Python harness, so it cannot yet be enforced there as a first-class CI metric.

This minimum applies to the repository-local automated coverage gate executed in CI.

## Enforcement scope

The 90% gate is enforced directly in repositories that already run executable unit or integration suites in CI:

- `backoffice`
- `api-gateway`
- `bff-mobile`
- `bff-backoffice`
- `microservice-users`
- `microservice-quizz`
- `microservice-wordpass`
- `ai-engine`

## Non-executable or not-yet-instrumented scopes

Some repositories do not currently expose a measurable unit/integration coverage surface in CI:

- documentation-only or validation-only repositories
- packaging-only repositories without test suites
- mobile scopes that still need dedicated coverage instrumentation work

Those scopes are still subject to the policy at platform level, but they are not yet fully enforced by an executable coverage gate.

## Operational rule

Coverage is a release gate, not an informational metric only.

Changes that reduce coverage below 90% in enforced scopes must fail CI until tests are expanded or code is reduced.

Repositories must expose the richest supported metric set their tooling can provide. When a stack cannot report a metric directly, the documentation must state that limitation explicitly instead of implying parity that does not exist.

## Reporting rule

Coverage baseline reports in central docs must distinguish:

1. measured current coverage
2. enforced threshold
3. whether the repository is currently compliant