# Target Architecture

## Objetivo

Definir la arquitectura objetivo para soportar clientes mobile y backoffice sobre una plataforma de microservicios y capacidades AI.

## Capas principales

1. Edge layer: `api-gateway` como punto de entrada publico.
2. Experience layer: `bff-mobile` y `bff-backoffice` con contratos optimizados por canal.
3. Domain services: microservicios de negocio (`microservice-users`, `microservice-quizz`, `microservice-wordpass`, `ai-engine`).
4. Data and async: bases de datos por servicio y eventos para procesos asincronos.
5. Platform: `platform-infra`, `observability-platform`, `contracts-and-schemas`, `shared-sdk-client`.

## Principios

- Un unico ingreso publico.
- APIs internas privadas.
- Contratos versionados y trazables.
- Observabilidad desde el inicio.
- Seguridad por defecto (least privilege, secretos externos, mTLS opcional por fase).
