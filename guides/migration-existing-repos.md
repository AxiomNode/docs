# Migration Of Existing Repositories To Gateway And BFF

## Objetivo

Adaptar los repos ya desarrollados a la estructura objetivo sin interrumpir el funcionamiento actual.

## Repos adaptados

- microservice-quizz
- microservice-wordpass
- microservice-users
- ai-engine

## Estrategia incremental

1. Mantener compatibilidad de endpoints actuales en microservicios.
2. Definir contratos internos en `contracts-and-schemas`.
3. Consumir microservicios desde BFFs y exponer solo gateway al exterior.
4. Deshabilitar gradualmente acceso directo externo a microservicios.

## Cambios operativos recomendados

- Ejecutar microservicios en red privada del clúster.
- Permitir entrada externa solo por `api-gateway`.
- Mantener docs privadas en microservicios para soporte interno.
- Consolidar auth de borde en gateway y auth contextual en BFF.

## Criterio de salida de migracion

- Mobile y backoffice consumen unicamente rutas de gateway.
- No hay clientes externos consumiendo microservicios de dominio directamente.
- Contratos versionados y publicados para cada servicio.
