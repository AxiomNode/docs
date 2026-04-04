# Target Architecture

## Goal

Define the target architecture for mobile and backoffice clients on top of a microservice-based platform with AI capabilities.

## Main layers

1. Edge layer: `api-gateway` as the single public entry point.
2. Experience layer: `bff-mobile` and `bff-backoffice` with channel-optimized contracts.
3. Domain services: `microservice-users`, `microservice-quizz`, `microservice-wordpass`, `ai-engine`.
4. Data and async: service-owned databases and event flows for asynchronous processing.
5. Platform layer: `platform-infra`, `observability-platform`, `contracts-and-schemas`, `shared-sdk-client`, `secrets`.

## Architecture principles

- Single public ingress.
- Private internal APIs by default.
- Versioned and traceable contracts.
- Observability-first runtime model.
- Secure-by-default posture (least privilege, externalized secrets, phased hardening).
