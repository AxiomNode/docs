# Common Code Extraction Audit (2026-03-24)

Last reviewed: 2026-04-19.

## Scope

Repositories audited for duplicated models, contracts, and shared logic:

- ai-engine
- api-gateway
- bff-mobile
- bff-backoffice
- microservice-quizz
- microservice-wordpass
- microservice-users
- shared-sdk-client
- contracts-and-schemas
- platform-infra

## Key Findings

- `microservice-quizz` and `microservice-wordpass` contained duplicated implementations of:
  - ai engine client
  - trivia categories/languages catalogs
  - private docs auth logic
- `microservice-users` shared the same private docs auth pattern.
- Shared repositories existed but were underused for these modules.

## Applied Extractions

### Shared SDK (`shared-sdk-client/typescript`)

Added new reusable modules:

- `src/ai-engine-client.ts`
- `src/trivia-categories.ts`
- `src/private-docs.ts`

Updated package exports to expose:

- `@axiomnode/shared-sdk-client/ai-engine-client`
- `@axiomnode/shared-sdk-client/trivia-categories`
- `@axiomnode/shared-sdk-client/private-docs`

### Contracts (`contracts-and-schemas`)

Added schema:

- `schemas/json/game-categories.v1.json`

### Service Adaptations

`microservice-quizz`:

- `src/app/services/aiEngineClient.ts` now re-exports shared module
- `src/app/services/triviaCategories.ts` now re-exports shared module
- `src/app/plugins/privateDocs.ts` now delegates auth/token helpers to shared module

`microservice-wordpass`:

- `src/app/services/aiEngineClient.ts` now re-exports shared module
- `src/app/services/triviaCategories.ts` now re-exports shared module
- `src/app/plugins/privateDocs.ts` now delegates auth/token helpers to shared module

`microservice-users`:

- `src/app/plugins/privateDocs.ts` now delegates auth helper to shared module
- Preserves strict `PRIVATE_DOCS_TOKEN` requirement (no AI key fallback)

## Validation Performed

- `shared-sdk-client/typescript`: `npm run build` passed
- `microservice-quizz/src`: `npm run build` passed
- `microservice-wordpass/src`: `npm run build` passed
- `microservice-users/src`: `npm run build` passed
- Private docs tests passed:
  - `microservice-quizz/src/tests/privateDocs.test.ts`
  - `microservice-wordpass/src/tests/privateDocs.test.ts`
  - `microservice-users/src/tests/privateDocs.test.ts`

## Remaining Migration Candidates (not yet moved)

- generation service base abstractions (`generationService.ts`, `modelGenerationJob.ts`)
- service metrics base collector (`serviceMetrics.ts`)
- shared request/monitoring validators
- gateway/bff health and proxy shared helpers

## Current assessment (2026-04-19)

The original extraction pass improved reuse meaningfully, but the workspace still shows repeated patterns at higher abstraction levels.

### High-value remaining consolidation targets

#### 1. Game generation service foundations

`microservice-quizz` and `microservice-wordpass` still share substantial structural behavior around:

- generation orchestration
- persisted model retrieval
- history listing
- resilience rules for invalid stored rows
- ai-engine retry and timeout semantics

These should only be extracted if the resulting abstraction remains domain-safe. The main risk is over-generalizing game-specific validation rules.

#### 2. Service operational diagnostics shapes

Backoffice-oriented monitoring and runtime targeting have introduced repeated concepts across repos:

- service target descriptors
- runtime source state (`env` vs `override`)
- operator-facing diagnostics payloads

Some of these may belong in shared contracts rather than local ad hoc DTOs.

#### 3. Delivery-policy helpers in CI

The service repos now use the same logical rule for central dispatch:

- validate first
- dispatch only on `main`
- skip central image publication for docs-only changes

This is duplicated in workflow YAML today. It is acceptable operationally, but it is a maintainability hotspot if the rule evolves again.

### Extraction rule of thumb

Before extracting shared logic, verify all three conditions:

1. behavior is duplicated in at least two active repos
2. the abstraction will not erase domain-specific invariants
3. the extracted module can be versioned and tested independently

### Recommended next extraction wave

1. Introduce shared contracts for runtime service-target and diagnostics payloads.
2. Evaluate a shared base for game-generation persistence filtering, but only after documenting the domain differences between quiz and word-pass.
3. Review whether workflow templating or reusable actions are worth the complexity for dispatch gating.

## Phase 2 — Game Route Schemas Extraction (2026-03-29)

`microservice-quizz` and `microservice-wordpass` shared ~10 identical local Zod schemas
in their `src/app/routes/games.ts` files. These were extracted to the SDK.

### New SDK module

- `src/game-schemas.ts` — exports `BaseGenerateSchema`, `IngestDocumentSchema`, `IngestSchema`,
  `RandomModelsQuerySchema`, `HistoryQuerySchema`, `GenerationProcessParamsSchema`,
  `GenerationProcessQuerySchema`, `GenerationProcessesListQuerySchema`,
  `ManualHistoryEntrySchema`, `HistoryItemParamsSchema`, plus inferred types.

### Service adaptations

- `microservice-quizz/src/app/routes/games.ts` now imports all schemas from
  `@axiomnode/shared-sdk-client`. `GenerateSchema` is an alias for `BaseGenerateSchema`.
- `microservice-wordpass/src/app/routes/games.ts` now imports all schemas and extends
  `BaseGenerateSchema` with `letters: z.string().optional()`.

### Additional cleanups (same date)

- **`SERVICE_COMMUNICATION_TOKEN` removed** from `secrets/scripts/bootstrap-secrets.mjs`
  and `secrets/scripts/validate-secrets-sync.mjs` — was generated but never consumed at runtime.
- **`game-categories.v1.json`** cross-referenced with `trivia-categories.ts` via comments
  (JSON Schema defines shape, TS module holds static data).

### Validation

- `shared-sdk-client/typescript`: `npx tsc` passed, `dist/game-schemas.js` emitted.

## Notes

- Local-only file `microservice-users/src/.env` was not modified as part of migration.
- Build-generated `microservice-users/src/dist/...` output was not included in migration commits.
