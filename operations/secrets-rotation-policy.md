# Secrets Rotation Policy

Last updated: 2026-04-23.

Operational procedure to rotate shared secrets in AxiomNode. Complements
[`environments-and-secrets.md`](./environments-and-secrets.md) and
[`secrets-distribution-guide.md`](./secrets-distribution-guide.md).

## Baseline cadence

| Secret class | Minimum frequency | Additional trigger |
|---|---|---|
| Shared internal tokens (`API_TOKEN`, `INTERNAL_API_TOKEN`, `AI_ENGINE_*_API_KEY`) | 90 days | suspected leak, staff rotation |
| CI/CD tokens (`PLATFORM_INFRA_DISPATCH_TOKEN`, `CROSS_REPO_READ_TOKEN`, `SECRETS_CROSS_REPO_READ_TOKEN`) | 180 days | repo owner change, GH alert |
| `PRIVATE_DOCS_TOKEN` | 180 days | publication / depublication of private docs |
| External provider credentials (registry, commercial models) | per provider TOS or 365 days | provider-forced rotation |
| Package / artifact signing keys | 365 days | suspected compromise |

Any suspected leak (accidental commit, exposed log, secret-scanning alert,
staff offboarding with access) overrides the cadence and forces immediate
rotation within 24 h.

## Roles

- **Secrets owner** (maintainer of the `secrets` repo): performs the
  rotation, publishes the new value to `secrets/runtime/*.env.secrets` and
  notifies consumers.
- **Service owner** (per runtime repo): validates that their service starts
  with the new value and approves the cutover in their distribution.
- **Release captain** (rotating role, see retros): coordinates simultaneous
  cutovers when a token affects several services.

## Standard procedure

1. **Prepare**
   - Open an issue in `secrets` with label `rotation` and the list of
     affected services.
   - Confirm that the chosen window is outside ongoing deployments (review
     `docs/operations/cicd-workflow-map.md`).
2. **Generate the new value**
   - Use `secrets/scripts/bootstrap-secrets.mjs` with `--rotate <KEY>` to
     generate a new value per distribution (`dev`, `stg`, `pro`).
   - Validate with `secrets/scripts/validate-secrets-sync.mjs` and
     `validate-no-placeholders.mjs`.
3. **Publish to runtime**
   - Distribute to the VPS hosts as described in
     `secrets-distribution-guide.md` (rsync/ansible or equivalent) and to
     the local PC that deploys `ai-engine`.
   - In K8s: update the corresponding `Secret` (
     `kubectl create secret generic ... --dry-run=client -o yaml | kubectl apply -f -`).
4. **Reload services**
   - Restart the consuming service (rolling restart in K8s, `compose up -d`
     in dev/stg VPS). For shared tokens, follow the order:
     producers → consumers → gateway.
5. **Verify**
   - `/healthz` and `/readyz` green.
   - Minimum smoke E2E (login + 1 protected endpoint).
   - The metric `*_requests_total{status="401"}` shows no sustained spikes.
6. **Close**
   - Mark the rotation issue as `done` with date and commit hash in
     `secrets`.
   - Note any incident in the weekly retro.

## Emergency rotation (<24 h)

- Skip the strict order: rotate the compromised token first across all
  affected distributions and accept a brief window of 401/403 errors in
  secondary services.
- Invalidate the previous value wherever possible (provider revocation,
  deletion of the prior Secret).
- Audit the last 72 h of logs with
  `secrets/scripts/audit-hardcoded-secrets.mjs` and `gh code search`.
- Open a post-mortem using the template described in
  [`retrospective-process.md`](./retrospective-process.md#post-mortems).

## Continuous auditing

- `audit-hardcoded-secrets.mjs` runs weekly in the `secrets` CI and blocks
  merges when leaks are detected.
- The `security.yml` workflows (CodeQL + Trivy) in every runtime repo flag
  secret patterns in code and dependencies.
- GitHub secret-scanning + push protection must remain enabled in every
  repo.

## References

- [`environments-and-secrets.md`](./environments-and-secrets.md).
- [`secrets-distribution-guide.md`](./secrets-distribution-guide.md).
- ADR 0008 — DevSecOps + SSDLC.
- OWASP ASVS V2 (authentication) and V6 (secret management).
