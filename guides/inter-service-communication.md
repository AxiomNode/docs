# Inter-Service Communication

Last updated: 2026-04-23.

Resumen ejecutivo de los contratos de comunicación entre servicios de
AxiomNode. La fuente operacional detallada (rutas, puertos, headers) sigue en
[`service-communication.md`](./service-communication.md). Este documento
recoge las **reglas duras** que deben respetarse en cualquier nuevo servicio
o ruta.

## Topología permitida

| Origen | Destino permitido | Prohibido |
|---|---|---|
| Cliente (mobile, backoffice) | `api-gateway` (`:7005`) | Acceso directo a BFFs o servicios |
| `api-gateway` | `bff-mobile`, `bff-backoffice`, rutas internas a `ai-engine` | Acceso directo a `microservice-*` |
| `bff-mobile` | servicios de dominio (`microservice-quizz`, `microservice-wordpass`, `microservice-users`) | Llamar a otro BFF |
| `bff-backoffice` | servicios de dominio + `ai-engine` (admin/diagnóstico) | Llamar a otro BFF |
| `microservice-quizz`, `microservice-wordpass` | `ai-engine` (`api`, `stats`) | Llamadas cruzadas entre servicios de dominio |
| `ai-engine-api` | llama runtime | Acceso directo desde clientes externos |

## Sincronía y semántica

- **Sync por defecto**, vía HTTP/JSON. Los BFFs pueden orquestar múltiples
  llamadas internas pero deben devolver una sola respuesta cohesionada.
- **Async/jobs**: solo para tareas largas (training, batch eval). En ese caso
  el BFF responde 202 + `job_id` y expone `/jobs/{id}` para polling. No hay
  websockets entre servicios.
- **Idempotencia**: toda mutación expuesta en `api-gateway` debe aceptar
  cabecera `Idempotency-Key` y respetarla durante al menos 24 h.
- **Timeouts**: el BFF aplica timeout duro de 8 s para llamadas a dominio y
  20 s para llamadas a `ai-engine` (modelo). Cualquier exceso degrada con
  fallback documentado en el contrato.

## Cabeceras obligatorias

| Cabecera | Origen | Propósito |
|---|---|---|
| `X-Correlation-ID` | inyectada en `api-gateway` si falta | Trazabilidad extremo a extremo |
| `Authorization: Bearer <token>` | cliente / BFF | Autenticación pública o interna |
| `X-Internal-Token: <INTERNAL_API_TOKEN>` | `api-gateway` → BFF | Validación de origen confiable |
| `X-Distribution: dev|stg|pro` | `api-gateway` | Selección de ruta y políticas |

Detalles de tokens: ver
[`docs/operations/environments-and-secrets.md`](../operations/environments-and-secrets.md).

## Errores y contrato de respuesta

- Formato uniforme `{ "error": { "code": string, "message": string, "details"?: object } }`.
- Los BFFs traducen errores internos a códigos de negocio. Nunca filtran
  trazas, IDs internos ni nombres de servicio downstream.
- `api-gateway` añade `X-Correlation-ID` también en respuestas de error.

## Observabilidad mínima por hop

Cada servicio expone:

- `/healthz` (liveness barata, sin dependencias).
- `/readyz` (readiness, valida dependencias críticas).
- `/metrics` Prometheus con counter `<svc>_requests_total{route,status}` y
  histogram `<svc>_request_duration_seconds`.

Para `ai-engine` aplican además las métricas LLMOps definidas en ADR 0009 y
descritas en
[`ai-prompting-rules.md`](./ai-prompting-rules.md#telemetría-obligatoria).

## Cambios de contrato

1. Editar el OpenAPI del servicio (`contracts-and-schemas/` cuando aplique o
   el repo del propio servicio).
2. Validar con `validate-contracts` en CI.
3. Versionar la ruta (`/v1/...`, `/v2/...`); jamás romper `/v1` activo.
4. Anunciar el cambio en la retro semanal
   ([`docs/operations/retrospective-process.md`](../operations/retrospective-process.md)).

## Referencias

- [`service-communication.md`](./service-communication.md) — contrato detallado.
- [`docs/operations/environments-and-secrets.md`](../operations/environments-and-secrets.md).
- [`docs/operations/secrets-rotation-policy.md`](../operations/secrets-rotation-policy.md).
- ADR 0006 (platform-infra centralizado).
- ADR 0007 (tags inmutables).
