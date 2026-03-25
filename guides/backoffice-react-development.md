# Guia De Desarrollo Backoffice (React)

## Objetivo

Definir como construir la app web de backoffice para gestion, observabilidad y modificacion de datos en los servicios de AxiomNode, usando React como frontend principal.

## Alcance Funcional

El backoffice debe cubrir 3 capacidades:

1. Gestion: operaciones administrativas sobre usuarios, catalogos y contenido generado.
2. Observabilidad: metricas operativas y estado de servicios para soporte y operaciones.
3. Correccion manual: herramientas para ajustar o corregir datos cuando falle la automatizacion.

## Arquitectura Recomendada

Flujo principal:

1. React Backoffice -> api-gateway (edge)
2. api-gateway -> bff-backoffice
3. bff-backoffice -> microservice-users y microservicios administrativos habilitados por contrato

Endpoints ya disponibles en el flujo backoffice:

- GET /v1/backoffice/users/leaderboard
- GET /v1/backoffice/monitor/stats

## Principios De Diseno

1. UI orientada a tareas operativas (tablas, filtros, acciones por fila, trazabilidad).
2. No consumir microservicios directamente desde el frontend; siempre via gateway + BFF.
3. Contratos tipados desde shared-sdk-client para evitar drift entre frontend y backend.
4. Feature flags para habilitar modulos gradualmente sin bloquear despliegues.

## Stack Frontend Recomendado

- React 19+
- Vite
- TypeScript estricto
- Router: React Router
- Estado remoto: TanStack Query
- Formularios: React Hook Form + Zod
- UI: libreria de componentes con tokens de diseno (evitar estilos ad hoc por pantalla)
- Graficas: Recharts o ECharts

## Estructura Inicial Sugerida

src/
  app/
    router/
    providers/
  modules/
    dashboard/
    users/
    moderation/
    settings/
  shared/
    api/
    ui/
    hooks/
    utils/
  pages/
  styles/

## Modulos Minimos (Fase 1)

1. Dashboard Operativo:
- consumira GET /v1/backoffice/monitor/stats
- mostrara estado de salud, trafico, auth, gameplay, generacion y logs recientes

2. Leaderboard De Usuarios:
- consumira GET /v1/backoffice/users/leaderboard
- filtros por metric (won, score, played) y limite

3. Vista De Incidencias:
- lista de errores operativos y eventos anomalo detectados en metricas
- enlaces directos a runbooks y acciones de mitigacion

## Modulos De Gestion (Fase 2)

Estas operaciones requieren ampliar BFF + gateway, porque hoy no estan expuestas para backoffice:

1. Gestion de perfiles de usuario (activar, bloquear, marcar, anotar)
2. Gestion de eventos de juego y correcciones
3. Moderacion de contenido generado por IA
4. Administracion de catalogos (categorias, idiomas, limites)

## Requisitos De Seguridad

1. Autenticacion de operadores:
- recomendado: Firebase Auth + custom claims para rol admin

2. Autorizacion por rol:
- roles sugeridos: admin, operator, analyst
- acciones criticas (modificar/borrar) solo para admin

3. Trazabilidad:
- registrar actor, accion, timestamp, entidad y diff en cada cambio manual

4. Secretos:
- no usar secretos hardcodeados en frontend
- los secretos de integracion viven en secrets y se distribuyen via prepare-runtime-secrets.mjs

## Integracion De Entornos

Desarrollo local:

1. Levantar microservicios de dominio.
2. Inyectar secretos de dev desde repo secrets.
3. Levantar edge integration con platform-infra.
4. Ejecutar frontend en modo dev contra api-gateway local.

Variables frontend recomendadas:

- VITE_API_BASE_URL (ej: http://localhost:7005)
- VITE_FIREBASE_API_KEY
- VITE_FIREBASE_AUTH_DOMAIN
- VITE_FIREBASE_PROJECT_ID
- VITE_APP_ENV

## Calidad Y Testing

1. Unit tests para componentes y hooks criticos.
2. Integration tests para paginas con query cache y formularios.
3. E2E tests (Playwright) para flujos de dashboard y leaderboard.
4. Contract tests contra respuestas reales de gateway en CI.

## Definition Of Done Por Modulo

1. UI accesible y responsive.
2. Manejo de errores y estados vacios.
3. Logs de auditoria para acciones de cambio.
4. Tests automatizados minimos:
- unit + integration para frontend
- smoke E2E para flujo completo

## Plan Recomendado De Implementacion

Fase A (base tecnica):
- bootstrap React + TypeScript + routing + auth shell + layout base

Fase B (observabilidad):
- dashboard + leaderboard + filtros

Fase C (gestion):
- endpoints de escritura en BFF/gateway + UI de acciones administrativas

Fase D (hardening):
- auditoria, permisos por rol, performance y observabilidad frontend

## Entregables Esperados

1. App backoffice desplegable en entorno dev/stg.
2. Guia operativa para soporte.
3. Matriz de permisos por rol.
4. Suite de pruebas E2E para caminos criticos.