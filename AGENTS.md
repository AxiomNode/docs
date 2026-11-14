# AGENTS

## Repo purpose
Central architecture and operations documentation for AxiomNode.

## Key paths
- architecture/: current platform architecture and repository map
- guides/: implementation and integration guides
- operations/: deployment, environments, CI/CD, runbooks
- design/: current brand and UI assets

## Local commands
- No build required; validate links and consistency locally before publishing.

## CI/CD notes
- Docs now have a markdown link validation workflow; keep relative links resolvable.
- Docs should track real workflow behavior (service CI dispatches the `platform-infra` image build on merge; deploys are manual-only via `deploy.yaml` `workflow_dispatch`, and no staging host exists today).

## LLM editing rules
- Keep content concise, actionable, and in English.
- Prioritize current-state accuracy over historical narrative.
- Cross-link related docs when changing workflow/architecture sections.
- Do not add roadmap, backlog, audit-history, or retrospective-only documents here.
