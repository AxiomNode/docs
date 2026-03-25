# AxiomNode Documentation

Repositorio central de documentacion para la arquitectura, operacion y evolucion de AxiomNode.

## Estructura principal

Documentos clave disponibles:

- [Arquitectura objetivo](./architecture/target-architecture.md)
- [Mapa de repositorios](./architecture/repository-map.md)
- [Modelo de comunicacion de servicios](./guides/service-communication.md)
- [Migracion de repos existentes](./guides/migration-existing-repos.md)
- [Integracion local de edge layer](./guides/local-edge-integration.md)
- [Guia de desarrollo backoffice (React)](./guides/backoffice-react-development.md)
- [Guia de desarrollo app movil (Firebase)](./guides/mobile-app-development-firebase.md)
- [Estrategia de despliegue](./operations/deployment-strategy.md)
- [Entornos y secretos](./operations/environments-and-secrets.md)
- [Distribucion de secretos por repositorio](./operations/secrets-distribution-guide.md)
- [Fases de implementacion (archivo historico)](./roadmap/implementation-phases.md)

## Uso recomendado

1. Revisar arquitectura y mapa de repositorios.
2. Definir contratos en `contracts-and-schemas`.
3. Implementar servicios en `api-gateway` y BFFs.
4. Desplegar con `platform-infra` y operar con `observability-platform`.

## Reglas de mantenimiento

- Mantener documentacion versionada junto a cambios de arquitectura.
- Evitar decisiones no documentadas entre repos.
- Mantener los documentos de roadmap como historial y registrar nuevas planificaciones en artefactos operativos vigentes.

## Diseno visual

Los assets de marca se mantienen en `design/`.
