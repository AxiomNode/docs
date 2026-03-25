# Guia De Desarrollo App Movil (Juegos IA + Firebase)

## Objetivo

Definir como construir la app movil de AxiomNode para consumir juegos generados por IA, autenticando usuarios con Firebase y registrando su actividad de juego.

## Vision Del Producto

La app movil debe permitir:

1. Registro e inicio de sesion con Firebase.
2. Descubrir contenido de quiz y word-pass.
3. Generar nuevas partidas con IA.
4. Guardar progreso, resultados y estadisticas del jugador.

## Arquitectura Recomendada

Flujo objetivo:

1. App movil -> api-gateway (edge)
2. api-gateway -> bff-mobile
3. bff-mobile -> microservice-quizz y microservice-wordpass
4. Para identidad y estadisticas: bff-mobile -> microservice-users

Endpoints de juegos ya disponibles por edge:

- GET /v1/mobile/games/quiz/random
- GET /v1/mobile/games/wordpass/random
- POST /v1/mobile/games/quiz/generate
- POST /v1/mobile/games/wordpass/generate

## Nota Importante De Seguridad

El EDGE_API_TOKEN es secreto de infraestructura. No debe embebirse en la app movil.

Para ambiente productivo, hay dos opciones validas:

1. api-gateway en modo publico para mobile con validacion de Firebase JWT server-side.
2. endpoint publico intermedio (BFF publico) que inyecte credenciales internas sin exponer secretos en cliente.

## Stack Tecnologico Recomendado

- React Native + Expo
- TypeScript estricto
- Navegacion: React Navigation
- Estado remoto: TanStack Query
- Estado local UI: Zustand o Context ligero
- Formularios: React Hook Form + Zod
- Almacenamiento seguro: expo-secure-store
- Telemetria de cliente: Sentry o equivalente

## Estructura Inicial Sugerida

src/
  app/
    navigation/
    providers/
  modules/
    auth/
    home/
    quiz/
    wordpass/
    profile/
    stats/
  shared/
    api/
    firebase/
    ui/
    hooks/
    utils/

## Flujo De Autenticacion Con Firebase

1. El usuario inicia sesion en Firebase (email/password, Google u otro provider).
2. La app obtiene idToken de Firebase.
3. La app envia idToken al backend para sincronizar sesion de dominio.

Endpoint de dominio ya implementado en users:

- POST /users/firebase/session

Tambien existen endpoints autenticados de perfil y stats:

- GET /users/me/profile
- GET /users/me/stats
- POST /users/me/games/events

## Gap Actual A Resolver Para Mobile Completo

Hoy esos endpoints de users no estan expuestos en bff-mobile/api-gateway.

Agregar en proxima iteracion:

1. Rutas en bff-mobile para session/perfil/stats/eventos.
2. Rutas proxy equivalentes en api-gateway.
3. Propagacion segura del bearer token Firebase hasta microservice-users.

## Contratos Y Payloads

Para generar juegos, categoryId es string (ejemplos validos: "23", "27").

Payload base para generate:

- language: string
- categoryId: string
- topic: string opcional
- numQuestions: number opcional

## Experiencia De Usuario Minima (MVP)

1. Onboarding + login Firebase.
2. Home con recomendaciones (random quiz/word-pass).
3. Pantalla de juego quiz.
4. Pantalla de juego word-pass.
5. Resultado y registro de evento en users.
6. Perfil con resumen de progreso.

## Estrategia De Datos Y Cache

1. Cache corta para random games (evitar repeticion inmediata).
2. Reintentos con backoff en generate por latencia de IA.
3. Fallback UX cuando generate tarde mas de lo esperado.
4. Persistencia local de ultima partida y progreso parcial.

## Errores Y Resiliencia

Manejar explicitamente:

1. 401 Unauthorized (token invalido o expirado).
2. 429 rate limit (si aplica en edge).
3. 502/504 de generacion IA (mostrar reintentar y usar contenido random cacheado).
4. Perdida de red (modo degradado con estado local).

## Entornos Y Configuracion

Variables recomendadas en app:

- EXPO_PUBLIC_API_BASE_URL
- EXPO_PUBLIC_FIREBASE_API_KEY
- EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN
- EXPO_PUBLIC_FIREBASE_PROJECT_ID
- EXPO_PUBLIC_FIREBASE_APP_ID
- EXPO_PUBLIC_APP_ENV

No incluir secretos internos de infraestructura en variables publicas.

## Testing Recomendado

1. Unit tests para logica de dominio cliente (transformaciones, hooks).
2. Integration tests para pantallas clave (auth, generate, resultado).
3. E2E (Detox o Maestro) para login + jugar + guardar evento.
4. Contract tests de cliente contra respuestas reales de edge en dev.

## Plan De Implementacion De Referencia

Fase A (core app):
- bootstrap Expo + navigation + auth state

Fase B (juego):
- random + generate quiz/word-pass + experiencia de juego

Fase C (identidad y progreso):
- session Firebase con users + perfil + stats + game events

Fase D (hardening):
- telemetria, crash reporting, performance, offline parcial

## Entregables Esperados

1. App movil funcional en dev con login Firebase.
2. Flujos completos de juego quiz y word-pass.
3. Persistencia de estadisticas por usuario.
4. Pruebas E2E para caminos criticos de negocio.