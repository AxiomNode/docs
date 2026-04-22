# Retrospective Process

Last updated: 2026-04-23.

AxiomNode runs lightweight but recurrent retros to keep multi-repo platform
quality high and to stay aligned with the master syllabus.

## Cadence

| Type | Frequency | Target duration | Participants |
|---|---|---|---|
| Platform weekly retro | weekly (Friday) | 30 min | core team |
| Release retro | after every cut to `pro` | 45 min | core team + affected service owners |
| Post-mortem | within 5 days after a Sev1/Sev2 incident | 60 min | responder + service owners + secrets owner if applicable |
| Quarterly architecture retro | every 3 months | 90 min | core team + cross review |

## Weekly retro

Fixed structure (suggested timebox in parentheses):

1. **Key metrics (5 min)**
   - CI status per repo (green / red / flaky).
   - `ai-engine` coverage (90% gate).
   - LLMOps metrics (`ai_engine_llm_fallback_total`, `errors_total`,
     `latency_p95_seconds`).
   - Active alerts in `observability-platform`.
2. **Progress (5 min)** — what was closed since the previous retro,
   referencing commits / PRs / ADRs.
3. **Blockers (5 min)** — what is preventing progress and who unblocks it.
4. **Actions (10 min)** — at most 5 actions, each with owner and target
   date. They roll over to the backlog or the relevant issue tracker.
5. **Risks and learnings (5 min)** — record anything worth memorizing in the
   notes (see "Operational memory").

Notes are stored in `docs/operations/retros/<YYYY-MM-DD>.md` (create that
folder when the first retro is run). Keep a minimum structure: `Metrics`,
`Progress`, `Blockers`, `Actions`, `Risks`.

## Release retro

- Review the immutable tags published (ADR 0007).
- Confirm `cicd-workflow-map.md` still matches reality.
- Validate that no secrets were left rotated halfway
  ([`secrets-rotation-policy.md`](./secrets-rotation-policy.md)).
- Record new issues detected during the rollout.

## Post-mortems

Recommended minimum template:

```
# Post-mortem <title>
- Incident date:
- Severity: Sev1 | Sev2 | Sev3
- Affected services:
- Detection (who and when?):
- Timeline (UTC):
- Root cause:
- Quantified impact:
- Immediate actions:
- Prevention actions (with owner and date):
- Lessons learned:
```

Rules:

- **Blameless**: discuss systems and processes, not people.
- **Measurable actions**: every prevention action ends in a PR, ADR or
  concrete operational change.
- **Visibility**: the post-mortem is stored in
  `docs/operations/post-mortems/<YYYY-MM-DD>-<slug>.md`.

## Quarterly architecture retro

- Review `docs/architecture/master-alignment.md` and move closed gaps into
  "Recent progress".
- Evaluate whether any ADR needs to be superseded.
- Confirm the repo list in `repository-map.md` is still complete.
- Decide the next batch of improvements to tackle.

## Operational memory

One-off decisions or learnings with long-term value are stored in:

- ADRs (when there is a structural decision).
- `docs/operations/*.md` (when they are operational policies).
- Internal notes in `docs/next-steps/` when they are short-term follow-ups.

If a retro action implies changes across several repos, open coordinated
issues and reference them from the retro document.

## References

- [`environments-and-secrets.md`](./environments-and-secrets.md).
- [`secrets-rotation-policy.md`](./secrets-rotation-policy.md).
- [`cicd-workflow-map.md`](./cicd-workflow-map.md).
- `docs/architecture/master-alignment.md`.
