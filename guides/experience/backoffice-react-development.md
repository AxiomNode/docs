# Backoffice Development Guide (React)

Last updated: 2026-05-03.

## Purpose

Describe the current-state development model for the AxiomNode backoffice web application.

## Scope

Use this guide for repository-local frontend development: source layout, UI constraints, local environment flow, and test commands.

Use the capability dossiers for operator diagnostics behavior, cross-repo route chains, and operational semantics.

## Current implementation shape

Current browser bootstrap:

- `backoffice/src/src/main.tsx`: mounts the SPA root.
- `backoffice/src/src/App.tsx`: handles auth gating, theme selection, typography preferences, and top-level layout selection.
- `backoffice/src/src/ui/`: operator-facing layouts, panels, and screen composition.
- `backoffice/src/src/application/`: application services such as session storage helpers.
- `backoffice/src/src/domain/`: UI-facing types and constants.
- `backoffice/src/src/infrastructure/`: runtime integration code.
- `backoffice/src/src/i18n/`: UI language context.

## Design principles

1. Task-oriented UI (tables, filters, contextual actions, auditability).
2. No direct frontend calls to domain microservices.
3. Distinguish persisted runtime state from client-side preferences.
4. Keep operator controls explicit and reversible.

## Current frontend stack

- React 18
- Vite
- Strict TypeScript
- Firebase authentication support
- Tailwind CSS
- Vitest + Testing Library
- Playwright for end-to-end coverage

## Current structure

```text
backoffice/src/src/
  App.tsx
  application/
  auth.ts
  data/
  domain/
  i18n/
  infrastructure/
  main.tsx
  runtimeConfig.ts
  ui/
```

## Security baseline

- Operator auth through Firebase (or equivalent) with role claims.
- Role-based authorization for privileged actions.
- Auditable manual changes (actor, action, timestamp, target, delta).
- No hardcoded secrets in frontend code.

Detailed operator-facing diagnostics behavior belongs in the operations capability docs rather than this repository-local guide.

## Local environment flow

1. From `secrets`, prepare development secrets.
2. From `platform-infra/environments/dev`, start the edge integration stack if integrated testing is needed.
3. From `backoffice/src`, run `npm install` and `npm run dev`.
4. Validate that the SPA points to the intended edge URL through `VITE_API_BASE_URL`.

## Testing baseline

- `npm run typecheck` for static verification.
- `npm run test:run` for Vitest-based automated tests.
- `npm run build` for production build validation.
- `npm run test:e2e` for Playwright-based operator flow checks.

## Related documents

- [Backoffice operational diagnostics](../capabilities/operations/backoffice-operational-diagnostics.md)
- [Runtime routing control](../capabilities/operations/runtime-routing-control.md)
- Repository-local references: `backoffice/docs/architecture/README.md` and `backoffice/docs/operations/README.md`