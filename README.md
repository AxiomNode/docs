# AxiomNode Documentation

Ultima actualizacion: 2026-03-29.

Repositorio central de documentacion para la arquitectura, operacion y evolucion de AxiomNode.

## Estructura principal

### Arquitectura

- [Arquitectura objetivo](./architecture/target-architecture.md) — capas, principios y flujo general.
- [Mapa de repositorios](./architecture/repository-map.md) — puertos, responsabilidades, dependencias y modulos SDK.
- [Auditoria de codigo comun](./architecture/common-code-extraction-audit-2026-03-24.md)

### Guias

- [Modelo de comunicacion de servicios](./guides/service-communication.md) — endpoints, auth entre capas, headers propagados.
- [Integracion local de edge layer](./guides/local-edge-integration.md)
- [Guia de desarrollo backoffice (React)](./guides/backoffice-react-development.md)
- [Guia de desarrollo app movil (Firebase)](./guides/mobile-app-development-firebase.md)
- [Migracion de repos existentes](./guides/migration-existing-repos.md)

### Operaciones

- [Entornos y secretos](./operations/environments-and-secrets.md) — variables por servicio, tokens compartidos, variable huerfana.
- [Distribucion de secretos](./operations/secrets-distribution-guide.md) — flujo de inyeccion, reglas de sincronizacion.
- [Estrategia de despliegue](./operations/deployment-strategy.md)
- [Runbook de servicios AI](./operations/ai-services-recovery-runbook.md)

### Referencia historica

- [Fases de implementacion](./roadmap/implementation-phases.md) — fases ya ejecutadas.

## Uso recomendado

1. Revisar arquitectura y mapa de repositorios.
2. Definir contratos en `contracts-and-schemas`.
3. Generar tipos con `npm run generate:contracts` en `shared-sdk-client/typescript`.
4. Implementar servicios consumiendo modulos del SDK.
5. Desplegar con `platform-infra` y operar con `observability-platform`.

## Reglas de mantenimiento

- Mantener documentacion versionada junto a cambios de arquitectura.
- Evitar decisiones no documentadas entre repos.
- Registrar cambios de contratos en `contracts-and-schemas` y regenerar SDK.

## Diseno visual

Los assets de marca se mantienen en `design/`.
