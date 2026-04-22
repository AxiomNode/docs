# Architecture Decision Records (ADRs)

ADRs document significant architectural decisions: context, alternatives considered, decision, and consequences.

Format: each file is `NNNN-short-title.md`, numbered sequentially. Status is one of `Proposed`, `Accepted`, `Superseded by NNNN`, or `Deprecated`.

## Index

| ID | Title | Status |
|---|---|---|
| [0001](./0001-single-public-edge.md) | Single public edge via api-gateway | Accepted |
| [0002](./0002-bff-per-channel.md) | BFF per channel (mobile / backoffice) | Accepted |
| [0003](./0003-contracts-as-source-of-truth.md) | Contracts repository as source of truth | Accepted |
| [0004](./0004-split-ai-runtime.md) | Split AI runtime (services in-cluster, llama optional) | Accepted |
| [0005](./0005-runtime-control-plane.md) | Backoffice-driven runtime control plane | Accepted |
| [0006](./0006-centralized-platform-infra.md) | Centralized build/deploy via platform-infra | Accepted |
| [0007](./0007-immutable-image-tags.md) | Immutable image tags for staging and prod | Accepted |
| [0008](./0008-devsecops-and-ssdlc.md) | DevSecOps and SSDLC as platform security baseline | Accepted |
| [0009](./0009-llmops-and-ai-evaluation.md) | LLMOps and eval harness in ai-engine | Accepted |

## Authoring rules

- Write the ADR before merging the architectural change it describes.
- Keep ADRs short (one page where possible).
- Never edit accepted ADRs; supersede them with a new ADR if the decision changes.
