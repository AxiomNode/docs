# AxiomNode Documentation

Repositorio central de documentacion para la arquitectura, operacion y evolucion de AxiomNode.

## Estructura principal

Documentos clave disponibles:

- [Arquitectura objetivo](./architecture/target-architecture.md)
- [Mapa de repositorios](./architecture/repository-map.md)
- [Modelo de comunicacion de servicios](./guides/service-communication.md)
- [Estrategia de despliegue](./operations/deployment-strategy.md)
- [Entornos y secretos](./operations/environments-and-secrets.md)
- [Fases de implementacion](./roadmap/implementation-phases.md)

## Uso recomendado

1. Revisar arquitectura y mapa de repositorios.
2. Definir contratos en `contracts-and-schemas`.
3. Implementar servicios en `api-gateway` y BFFs.
4. Desplegar con `platform-infra` y operar con `observability-platform`.

## Reglas de mantenimiento

- Mantener documentacion versionada junto a cambios de arquitectura.
- Evitar decisiones no documentadas entre repos.
- Actualizar roadmap al cerrar cada fase.

## Diseno visual

Los assets de marca se mantienen en `design/`.
