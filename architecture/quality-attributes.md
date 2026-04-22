# Quality Attribute Scenarios

Last updated: 2026-04-22.

Each scenario uses the SEI template (Source, Stimulus, Environment, Artifact, Response, Response measure). Scenarios are descriptive of the current target, not aspirational beyond what platform-infra and observability-platform can currently observe.

## Performance

### QA-P1 Quiz round latency under nominal load

- Source: Mobile player.
- Stimulus: `GET /v1/games/quiz/random` with cached generation available.
- Environment: staging or production, normal load (<= 50 RPS per service).
- Artifact: `api-gateway` -> `bff-mobile` -> `microservice-quizz`.
- Response: returns 200 with cached `GameGeneration`.
- Measure: p95 < 400 ms end-to-end; cache hit ratio > 70 % observed via stats.

### QA-P2 AI cold generation latency

- Source: Internal service (cache miss).
- Stimulus: `POST /generate` with no matching cache key.
- Environment: split-AI runtime, llama reachable.
- Artifact: `ai-engine-api` + llama runtime.
- Response: returns 200 generation.
- Measure: p95 < 6 s with `paraphrase-multilingual-MiniLM-L12-v2` and current corpus.

## Availability

### QA-A1 Edge degraded gracefully when one BFF is down

- Source: Mobile player.
- Stimulus: `bff-mobile` unhealthy.
- Environment: staging.
- Artifact: `api-gateway`.
- Response: returns 503 with stable error envelope and correlation id; backoffice channel unaffected.
- Measure: error envelope conformance 100 %; backoffice success rate unchanged.

### QA-A2 AI subsystem failure does not affect non-AI flows

- Source: Operator or external.
- Stimulus: `ai-engine-api` or llama unreachable.
- Environment: any.
- Artifact: gameplay services with cached generations.
- Response: cached games keep serving; uncached requests fail with explicit AI-unavailable error.
- Measure: non-AI gameplay error rate unchanged; AI-dependent error rate < 100 % thanks to cache hit ratio.

## Security

### QA-S1 No public endpoint reaches a domain service directly

- Source: External attacker.
- Stimulus: HTTP request to `/microservice-*` host.
- Environment: staging/prod.
- Artifact: ingress + NetworkPolicy.
- Response: connection refused; only `api-gateway` is publicly routable.
- Measure: external port scan finds only gateway port; verified manually.

### QA-S2 Pods cannot escalate privileges

- Source: Compromised container.
- Stimulus: attempt to gain root or mount sensitive paths.
- Environment: any.
- Artifact: pod security context.
- Response: `runAsNonRoot=true`, `allowPrivilegeEscalation=false`, `seccompProfile=RuntimeDefault`, `automountServiceAccountToken=false`.
- Measure: 11/11 base deployments compliant (audit 2026-04-07).

### QA-S3 No mutable image tag in production

- Source: Platform engineer.
- Stimulus: deploy attempt referencing `:latest`.
- Environment: CI.
- Artifact: `validate-infra` workflow guard.
- Response: build fails before reaching k3s.
- Measure: 0 occurrences of `:latest` in any base or overlay manifest.

## Maintainability

### QA-M1 Contract change blocks drifting consumers

- Source: Contracts maintainer.
- Stimulus: breaking change merged to `contracts-and-schemas`.
- Environment: CI of consumers.
- Artifact: SDK regeneration step.
- Response: dependent service builds fail until contracts are honored.
- Measure: 100 % of consumer pipelines re-run on SDK update.

### QA-M2 Service repo can be regenerated to a known good baseline

- Source: New engineer.
- Stimulus: clone + `npm install` + `npm run build` (or equivalent).
- Environment: clean machine.
- Artifact: per-repo `README.md`.
- Response: green local build with documented commands.
- Measure: minimum-standards compliance reported in `repository-minimum-standards.md`.

## Operability

### QA-O1 Routing change without redeploy

- Source: Operator.
- Stimulus: AI target change via backoffice.
- Environment: any.
- Artifact: `bff-backoffice` + `api-gateway` + `ai-engine-api` PV state.
- Response: change takes effect within seconds and survives pod restart.
- Measure: change reflected in next health response; observable in stats.

### QA-O2 Recovery runbook completion time

- Source: Platform engineer.
- Stimulus: AI services degraded.
- Environment: staging.
- Artifact: `ai-services-recovery-runbook.md`.
- Response: operator follows runbook to restore AI flow.
- Measure: target MTTR < 30 minutes including switch to workstation-hosted llama if needed.

## Evolvability

### QA-E1 New domain service can be added without touching existing services

- Source: Engineering team.
- Stimulus: introduce a new game type.
- Environment: workspace.
- Artifact: new microservice repo + contracts entry + SDK regen.
- Response: new BFF route + gateway forward; no schema changes to other services.
- Measure: 0 modifications required to existing microservice DBs.

## Cost / efficiency

### QA-C1 AI cache reduces llama spend

- Source: AI engine.
- Stimulus: repeated similar generation requests.
- Environment: any.
- Artifact: cache key (category + model + corpus signature).
- Response: served from cache.
- Measure: hit ratio tracked via `ai-engine-stats`; target > 60 %.

## Observability

### QA-Obs1 Every request is correlatable

- Source: Mobile or operator client.
- Stimulus: any request.
- Environment: any.
- Artifact: gateway + BFFs.
- Response: correlation id propagated end-to-end.
- Measure: correlation id present in 100 % of structured logs sampled.

## Cross-cutting non-functional matrix

| Attribute | Owner | Primary instrument |
|---|---|---|
| Performance | service repos + ai-engine | per-service metrics, ai-engine-stats |
| Availability | platform-infra | k3s probes + alerts (observability) |
| Security | platform-infra + secrets | NetworkPolicy, sealed-secrets, pod security context |
| Maintainability | contracts + sdk | CI gates and audit guards |
| Operability | bff-backoffice + api-gateway | runtime control plane |
| Evolvability | docs + contracts | architecture docs and contract versioning |
| Cost | ai-engine | cache hit metrics |
| Observability | observability-platform | dashboards + alert rules |

## Related documents

- [Target architecture](./target-architecture.md)
- [Use cases](./use-cases.md)
- [Operations runbook](../operations/ai-services-recovery-runbook.md)
