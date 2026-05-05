# Seed-data Preparation And Repair

Last updated: 2026-05-03.

## Purpose

This capability prepares initial quiz and word-pass datasets and supports bulk repair of stored content when inventory quality must be corrected.

## Functional objective

- generate importable seed data from external sources
- normalize and repair content after ingestion
- support operational data maintenance outside core runtime repos

## Ownership and boundaries

### Main owners

- `runtime-seed`

### Downstream dependencies

- external content sources such as OpenTrivia and FrequencyWords
- backoffice APIs for bulk repair paths

### Explicit boundaries

- owns dataset generation and repair tooling outside runtime repos
- delegates persistent storage to downstream services and their APIs
- does not own the runtime serving path for generated content

## Entrypoints and contracts

### Main entrypoints

| Type | Location | Notes |
|---|---|---|
| Script | `generate_seed.py` | seed content preparation |
| Artifact | CSV variants such as `quiz_seed.csv` and `wordpass_seed.csv` | bulk import inputs |
| Script | `fix_wordpass_remote.py`, `fix_wordpass_fast.py`, `clean_history_remote.py` | corrective maintenance |

### Inputs

- external source datasets
- environment tokens and API base URLs for repair workflows
- cleanup and sizing parameters

### Outputs

- CSV seed files
- repaired content through API side effects
- script logs and import-ready artifacts

### Contract sources

- repair payload shapes exposed through backoffice and service data endpoints
- CSV import format expectations of quiz and word-pass services

## Main functional flow

1. External content is fetched or read by the seed scripts.
2. The scripts normalize it into the import shapes expected by the runtime services.
3. CSV artifacts are produced for import or bulk seeding.
4. Repair scripts patch existing persisted records through protected backoffice or service APIs when content quality is insufficient.

## Persistence and runtime state

### Owned persistence

- generated CSV artifacts and repair scripts

### Runtime state

- affected persisted records in downstream services after import or repair
- repair paths depend on current protected backoffice APIs and current schema expectations

### State risks

- source data and repaired records can drift
- repair scripts can encode assumptions that no longer match live schemas
- repeated bulk runs can mutate history unexpectedly

## Observability evidence

### Health signals

- successful API reachability for repair flows

### Operational signals

- script output
- backoffice API responses
- post-import and post-repair data inspection

### Verification evidence

- sample validation of produced CSVs
- inventory checks after import
- operator comparison before and after repair

## Failure modes and risks

| Failure mode | Likely cause | User/operator impact | Detection path | Recovery path |
|---|---|---|---|---|
| Bad source quality | noisy external data | low-quality inventory | sampling and validation checks | regenerate, filter, or repair data |
| Accidental mass update | bad repair parameterization | widespread content corruption | data diff and API responses | restore from known-good source and rerun safely |
| Script/schema mismatch | repair tooling lags behind live APIs | bulk repair fails or corrupts records | failed repair runs and post-run inspection | align scripts with current APIs before rerun |

## Security and access model

- repair scripts use privileged edge/backoffice access and should be handled as admin tooling
- external source ingestion should not bypass validation or introduce unsafe content blindly

## Deployment and rollback implications

### Deployment impact

- data repair is operationally separate from service rollout but can change user-visible behavior immediately
- import/repair activities should be coordinated with current schema expectations

### Rollback impact

- rollback may require restoring data, not only redeploying code
- reversing a bad repair is a data-operations task rather than a binary rollback

## Related capabilities

- [Quiz inventory and generation](../domain/quiz-inventory-and-generation.md)
- [Word-pass inventory and generation](../domain/wordpass-inventory-and-generation.md)
- [Contract governance](./contract-governance.md)

## Open questions

- whether seed and repair tooling should have stronger formal runbooks and dry-run modes
- whether all current import formats are documented centrally enough to survive schema changes

## References

- [Platform capability matrix](../../foundations/platform-capability-matrix.md)
- [Word-pass inventory and generation](../domain/wordpass-inventory-and-generation.md)
- [Quiz inventory and generation](../domain/quiz-inventory-and-generation.md)
