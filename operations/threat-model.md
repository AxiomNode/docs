# Threat Model — STRIDE baseline

Last updated: 2026-04-23.

First formal threat-modeling exercise for AxiomNode. Covers the runtime
services and the supporting repositories. Applies STRIDE (Spoofing,
Tampering, Repudiation, Information disclosure, Denial of service,
Elevation of privilege) per component and ranks each row with a qualitative
score (low / medium / high). Living document: every relevant architecture
change must update the affected section and open a follow-up issue if a new
mitigation appears.

Reference sources:

- ADR 0006 — centralized platform-infra.
- ADR 0007 — immutable tags.
- ADR 0008 — DevSecOps + SSDLC.
- ADR 0009 — LLMOps and eval harness.
- [`docs/guides/inter-service-communication.md`](../guides/inter-service-communication.md).
- [`docs/guides/ai-prompting-rules.md`](../guides/ai-prompting-rules.md).
- [`docs/operations/secrets-rotation-policy.md`](./secrets-rotation-policy.md).
- [`docs/operations/environments-and-secrets.md`](./environments-and-secrets.md).

## Scope and assumptions

- **Runtime services**: `api-gateway`, `bff-mobile`, `bff-backoffice`,
  `microservice-quizz`, `microservice-wordpass`, `microservice-users`,
  `ai-engine` (including the llama runtime), `backoffice`, `mobile-app`.
- **Support repositories**: `secrets`, `platform-infra`,
  `observability-platform`, `contracts-and-schemas`, `shared-sdk-client`,
  `docs`.
- **Distributions**: `dev` (developer PC + dev VPS), `stg` (VPS),
  `pro` (VPS + local ai-engine).
- **Trust assumptions**:
  - The identity provider (Firebase Auth) is trusted.
  - The image registry (GHCR) is protected by a rotated token.
  - GitHub branch protection and `security.yml` (CodeQL + Trivy) are active
    on all 8 runtime repos.
  - The operator with access to the PC that deploys `ai-engine` is internal.

## Trust boundaries

```
[ Mobile client / browser backoffice ]
              | (HTTPS, JWT)
              v
        [ api-gateway :7005 ]   <-- public boundary
              |  (HTTPS + INTERNAL_API_TOKEN)
        +-----+---------------------+
        v                           v
  [ bff-mobile ]              [ bff-backoffice ]
        |                           |
        +------+--------+-----------+
               v        v
   [ microservice-quizz, -wordpass, -users ]   [ ai-engine-api ]
                                                       |
                                                       v
                                              [ llama runtime ]
```

Critical boundaries:

1. Internet ↔ `api-gateway` (authentication + quotas).
2. `api-gateway` ↔ BFFs (internal token + origin headers).
3. BFFs ↔ domain services (per-distribution shared token).
4. `ai-engine-api` ↔ llama runtime (local channel, not exposed to internet).
5. Any service ↔ `secrets` (via deployment artifacts, not at runtime).

## Assets

| Asset | Classification | Notes |
|---|---|---|
| User data (`microservice-users`) | PII | Requires encryption in transit and at rest |
| Internal tokens (`API_TOKEN`, `AI_ENGINE_*`, `INTERNAL_API_TOKEN`) | Confidential | Regulated rotation, see `secrets-rotation-policy.md` |
| Locally loaded LLM models | Confidential | Risk of exfiltration via internal endpoints |
| Prompts, RAG and eval datasets | Internal | May contain product IP |
| Logs and Prometheus metrics | Internal | May leak PII if not sanitized |
| Persisted runtime configuration (routes, presets) | Internal | Changes behavior without restart |

## STRIDE summary per component

Cells marked with `—` indicate a low-relevance threat or one already covered
by another component's mitigation.

### `api-gateway`

| STRIDE threat | Risk | Vector | Current mitigation | Pending actions |
|---|---|---|---|---|
| Spoofing | High | Stolen or replayed JWT | Signature validation, `Authorization: Bearer`, short expiration | Review TTL and refresh policy |
| Tampering | Medium | Payload modification in transit | HTTPS terminated at gateway/edge | Force HSTS and review internal mTLS to BFFs |
| Repudiation | Medium | Lack of audit on admin changes | `X-Correlation-ID` propagated, structured logs | Persist admin logs in a dedicated backend with retention >= 90 days |
| Information disclosure | Medium | Permissive CORS or stack-trace errors | CORS policy restricted per distribution | Quarterly CORS audit + review of error messages |
| Denial of service | High | Public traffic saturation | Hard timeouts documented in `inter-service-communication.md`, IP-based fixed-window rate limiter (`app/services/rateLimiter.ts`) with stricter quotas on generation/auth/admin categories, and `CircuitBreaker` (shared SDK) on every BFF/AI upstream | Promote in-memory limiter to a shared store (Redis) once horizontal scaling is enabled |
| Elevation of privilege | High | Leaked `X-Internal-Token` header | Token rotated through `secrets/scripts/bootstrap-secrets.mjs` | Move to a per-distribution signed token validated in BFFs |

### `bff-mobile` and `bff-backoffice`

| STRIDE threat | Risk | Vector | Current mitigation | Pending actions |
|---|---|---|---|---|
| Spoofing | High | Direct calls bypassing the gateway | `INTERNAL_API_TOKEN` required on every request | Block public binding on the VPS (internal overlay only) |
| Tampering | Medium | Manipulation of forwarded payload | Schema validation in BFF before upstream | Generate contracts from `contracts-and-schemas/` and fail CI on drift |
| Repudiation | Medium | Admin actions (backoffice) without traceability | Logs with `correlation_id` + operator role, append-only JSONL audit trail (`bff-backoffice/src/app/services/auditTrailStore.ts`) with daily rotation and 90-day retention exposed via `GET /v1/backoffice/admin/audit` | Replicate audit trail to a tamper-evident store (object lock / WORM) for long-term retention |
| Information disclosure | Medium | Internal metadata leaked to the client | Typed and sanitized responses | Add a negative contract test (no internal properties) |
| Denial of service | Medium | Concurrent calls to `ai-engine` | Timeouts (8 s domain / 20 s ai-engine) | Per-user queue/limit and declared backpressure |
| Elevation of privilege | High (backoffice) | Bypass of `roleHasBackofficeAccess` | Validation at login + server-side role | Add an E2E test that asserts 403 on admin routes without role |

### Domain microservices (`quizz`, `word-pass`, `users`)

| STRIDE threat | Risk | Vector | Current mitigation | Pending actions |
|---|---|---|---|---|
| Spoofing | Medium | Calls with another service's token | Per-distribution `API_TOKEN` | Move to per-service tokens once the model matures |
| Tampering | Medium | Injection in query parameters | Validation with Zod / Pydantic | Audit endpoints without validators and cover them with tests |
| Repudiation | Medium | Changes on user data without traceability | Structured logs | Add an `audit_log` table for mutations in `users` |
| Information disclosure | High (`users`) | Endpoints returning more PII than needed | Separate `Public*` schemas | Periodic review + Trivy fs / SAST cover the rest |
| Denial of service | Medium | Bucket without limit on public endpoints via gateway | Limit applied at gateway (planned) | Define per-endpoint SLO and alert in `observability-platform` |
| Elevation of privilege | Medium | Role check only in BFF | Partial server-side duplicate validation | Move authorization decisions to the microservice owning the resource |

### `ai-engine` (api + stats + llama runtime)

| STRIDE threat | Risk | Vector | Current mitigation | Pending actions |
|---|---|---|---|---|
| Spoofing | High | Reuse of `AI_ENGINE_API_KEY` | Per-service token + 90 d rotation | Sign requests with `correlation_id + nonce` (backlog) |
| Tampering | High | Prompt injection / RAG data poisoning | Rules in [`ai-prompting-rules.md`](../guides/ai-prompting-rules.md), input sanitization, schema validation, heuristic jailbreak classifier (`ai_engine.safety`) wired before the LLM | Replace heuristic classifier with a learned model once telemetry justifies it |
| Repudiation | Medium | Prompt changes without traceability | `PROMPT_VERSIONS` + versioned commits | Persist `prompt_version` on every response and expose it in metrics |
| Information disclosure | High | Leaking the system prompt or RAG docs | The system prompt forbids revealing instructions | Regression test that sends known exfiltration prompts |
| Denial of service | High | Huge prompts / generation loops | Maximum sizes, timeouts and `llm_fallback_total` | Prometheus alert if `rate(ai_engine_llm_fallback_total) > 0.1 / 5m` |
| Elevation of privilege | Medium | Admin endpoints (`/admin/*`) reachable without token | Specific token (`AI_ENGINE_BRIDGE_API_KEY`) | Bind the admin port to the internal interface |

### `backoffice` and `mobile-app`

| STRIDE threat | Risk | Vector | Current mitigation | Pending actions |
|---|---|---|---|---|
| Spoofing | Medium | Firebase session impersonation | Firebase Auth + server-side role in BFF | Force reauth on sensitive changes |
| Tampering | Medium | Manipulation of the served bundle | Immutable tags (ADR 0007) + registry integrity | Enable SRI on assets when applicable |
| Repudiation | Low | User actions without log | `correlation_id` sent on every request | — |
| Information disclosure | Medium | Client logs with PII | Minimal logging in production | Audit `console.log` before release |
| Denial of service | Low | Client able to saturate the gateway | Limits at gateway | — |
| Elevation of privilege | Medium | Token manipulation in localStorage | In-memory token + short refresh | Move to HttpOnly cookies once evaluated |

### Support: `secrets`, `platform-infra`, `observability-platform`

| STRIDE threat | Risk | Vector | Current mitigation | Pending actions |
|---|---|---|---|---|
| Spoofing | Medium | Malicious workflow impersonating a dispatch | `PLATFORM_INFRA_DISPATCH_TOKEN` rotated, every workflow declares an explicit top-level `permissions:` block, and the policy in [`github-actions-hardening.md`](./github-actions-hardening.md) requires GitHub Environments with reviewers for `stg`/`prod` deploys | Finish provisioning the Environment reviewer matrix in the GitHub UI as tracked in §5 of the hardening policy |
| Tampering | High | Commit with hardcoded secret | `audit-hardcoded-secrets.mjs` + push protection | Add a mandatory pre-commit hook in every repo |
| Repudiation | Medium | Infra changes without a clear owner | CODEOWNERS per folder | Keep up to date and review at retros |
| Information disclosure | High | Pipeline logs exposing secrets | `::add-mask::` and output review; multi-line PEM keys (`K3S_SSH_KEY`) are re-masked line by line in `platform-infra/.github/workflows/deploy.yaml` | Document in `docs/operations/cicd-workflow-map.md` which outputs are safe |
| Denial of service | Low | Infinite workflow | Per-job timeouts (GitHub default) | — |
| Elevation of privilege | High | Token with `repo` scope leaked | Per-purpose tokens (dispatch / read), most CI tokens already migrated to fine-grained PATs with the matrix documented in [`github-actions-hardening.md`](./github-actions-hardening.md) | Replace the remaining classic `GHCR_PULL_TOKEN` with a fine-grained PAT scoped to `packages:read` |

## Top risks and next iteration

1. **Prompt injection and exfiltration in `ai-engine`** (Tampering /
   Disclosure, high). Backlog: jailbreak classifier + regression suite of
   prompts. _Status 2026-04-23: heuristic classifier shipped in_ `ai_engine.safety` _with regression dataset; learned classifier remains future work._
2. **DoS at `api-gateway`** (DoS, high). Backlog: IP-based rate limiting and
   circuit breaker towards BFFs. _Status 2026-04-23: per-IP fixed-window rate
   limiter shipped in_ `api-gateway/src/app/services/rateLimiter.ts` _with
   stricter quotas for generation/auth/admin categories and Prometheus
   counters_ `gateway_rate_limit_blocks_total` _/_ `gateway_rate_limit_blocks_by_category_total`_; `CircuitBreaker` already in place on every BFF/AI upstream. Distributed-store upgrade remains future work._
3. **Audit trail of admin actions** (Repudiation, medium in BFFs and
   support). Backlog: centralized persistence with retention >= 90 days.
   _Status 2026-04-23: append-only JSONL audit trail shipped in_
   `bff-backoffice/src/app/services/auditTrailStore.ts` _with daily rotation,
   90-day retention, and a query endpoint at_ `GET /v1/backoffice/admin/audit`.
   _Tamper-evident replication remains future work._
4. **Rotation and least scope for CI/CD tokens** (Spoofing / EoP,
   medium-high). Backlog: migrate to fine-grained tokens and require
   `environments` with reviewers.
   _Status 2026-04-23: hardening policy published in_
   [`github-actions-hardening.md`](./github-actions-hardening.md) _with the
   Environment reviewer matrix, fine-grained token inventory, workflow
   standards, and a migration plan. `platform-infra/.github/workflows/build-push.yaml`
   and_ `deploy.yaml` _now ship annotated minimum `permissions:` blocks and
   re-mask the multi-line `K3S_SSH_KEY` line by line. Remaining work:
   provision the GitHub Environments in the UI, replace the classic_
   `GHCR_PULL_TOKEN` _with a fine-grained PAT, and roll out CODEOWNERS for_
   `.github/workflows/` _across every repo._

## Cadence and maintenance

- Quarterly review at the architecture retro
  ([`retrospective-process.md`](./retrospective-process.md)).
- Ad-hoc update when a new service is added or a trust boundary changes
  (publishing a new endpoint, connecting an external provider, etc.).
- Any pending action unresolved at 90 days is escalated to "accepted risk"
  or replanned in the next retro.
