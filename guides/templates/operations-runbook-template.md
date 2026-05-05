# Operations Runbook Template

Last updated: 2026-05-03.

## Purpose

Use this template for operational procedures, runbooks, and recovery guides that must be executed reliably under time pressure.

## Template

```md
# <Runbook or operational policy name>

Last updated: YYYY-MM-DD.

## Purpose

State what this runbook or policy is for.

## When to use this

- Triggering condition
- Incident symptom
- Planned maintenance scenario

## Preconditions

- Required access
- Required secrets or tokens
- Environment assumptions

## Execution steps

1. Step one.
2. Step two.
3. Step three.

## Validation

- Health checks
- Smoke checks
- Expected logs, metrics, or UI state

## Rollback or recovery

- Safe rollback boundary
- Reversal steps
- Escalation path if rollback is not enough

## Risks and cautions

- State drift
- Partial success cases
- Common operator mistakes

## References

- Link to relevant architecture, capability, and repository documents.
```

## Usage notes

- Keep steps ordered and executable.
- Prefer explicit validation and rollback sections over long narrative prose.