# Coverage Baseline Report

Last updated: 2026-04-22.

## Scope

This report summarizes the current measured or instrumented automated test-coverage state for AxiomNode executable repositories.

## Coverage gate target

- Required minimum in Vitest scopes: 90% for lines, statements, functions, and branches.
- Required minimum in Python coverage.py scopes: 90% combined total coverage, with branch instrumentation enabled and branch percentage tracked separately.

## Current baseline

The table below is updated from repository-local coverage runs performed during documentation and CI-hardening work.

| Repository | Stack | Gate | Lines | Statements | Functions | Branches | Status | Notes |
|---|---|---:|---:|---:|---:|---:|---|---|
| `backoffice` | Vitest + jsdom | 90% | 80.06% | 79.42% | 78.54% | 64.96% | noncompliant | Coverage gate configured; branch coverage is the limiting metric |
| `api-gateway` | Vitest | 90% | 97.95% | 97.95% | 100.00% | 90.65% | compliant | Coverage gate now passes locally after expanding `proxy.ts` route and state-store tests |
| `bff-mobile` | Vitest | 90% | 100.00% | 100.00% | 100.00% | 92.15% | compliant | Coverage gate now passes locally |
| `bff-backoffice` | Vitest | 90% | 94.65% | 94.65% | 97.56% | 90.06% | compliant | Coverage gate now passes locally after route-guard extraction, targeted branch tests, and narrowly scoped defensive-branch exclusions |
| `microservice-users` | Vitest | 90% | 97.36% | 97.36% | 100.00% | 91.51% | compliant | Coverage gate now passes locally after expanding route, auth, config, monitoring, metrics, private-docs, and user-service tests, plus narrowly scoped defensive-branch exclusions in authenticated routes |
| `microservice-quizz` | Vitest | 90% | 95.28% | 95.28% | 95.34% | 90.06% | compliant | Coverage gate now passes locally after expanding config, monitoring, private-docs, scheduler, Prisma singleton, games routes, metrics, and generation-service tests |
| `microservice-wordpass` | Vitest | 90% | 96.40% | 96.40% | 95.34% | 92.10% | compliant | Coverage gate now passes locally after expanding config, monitoring, private-docs, scheduler, Prisma singleton, games routes, metrics, and generation-service tests |
| `ai-engine` | pytest-cov | 90% combined total | 83.66% | 83.66% | n/a | 68.77% | noncompliant | Current non-gating run produced 80.37% combined total coverage with 7 failing tests; CI now enables branch instrumentation |
| `mobile-app` | Gradle/KMP | 90% policy target | n/a | n/a | n/a | n/a | pending instrumentation | Test suite exists but no coverage instrumentation is wired in CI |
| `shared-sdk-client/typescript` | TypeScript package | 90% policy target | n/a | n/a | n/a | n/a | noncompliant / pending suite | No automated test suite present |

## Notes

- Repositories without executable tests or without a coverage harness are not exempt from the platform rule; they are pending enforcement work.
- Baselines above were collected from local manual runs intended to measure the current state before full compliance work.
- `ai-engine` currently uses coverage.py semantics rather than Vitest semantics; documentation must not imply that function coverage is already emitted there or that branches are gated independently.
- Non-executable documentation, contracts, secrets, and infrastructure repositories are outside line/function/branch coverage scope and should be validated through their own structural checks instead.