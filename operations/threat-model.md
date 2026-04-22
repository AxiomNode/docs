# Threat Model — STRIDE baseline

Last updated: 2026-04-23.

Primer ejercicio formal de modelado de amenazas para AxiomNode. Cubre los
servicios runtime y los repositorios de soporte. Se aplica STRIDE
(Spoofing, Tampering, Repudiation, Information disclosure, Denial of
service, Elevation of privilege) por componente y se prioriza con un score
cualitativo (low / medium / high). Documento vivo: cada cambio relevante de
arquitectura obliga a actualizar la sección afectada y a abrir un issue de
seguimiento si surge una mitigación nueva.

Fuentes de referencia:

- ADR 0006 — platform-infra centralizado.
- ADR 0007 — tags inmutables.
- ADR 0008 — DevSecOps + SSDLC.
- ADR 0009 — LLMOps y eval harness.
- [`docs/guides/inter-service-communication.md`](../guides/inter-service-communication.md).
- [`docs/guides/ai-prompting-rules.md`](../guides/ai-prompting-rules.md).
- [`docs/operations/secrets-rotation-policy.md`](./secrets-rotation-policy.md).
- [`docs/operations/environments-and-secrets.md`](./environments-and-secrets.md).

## Alcance y supuestos

- **Servicios runtime**: `api-gateway`, `bff-mobile`, `bff-backoffice`,
  `microservice-quizz`, `microservice-wordpass`, `microservice-users`,
  `ai-engine` (incluido llama runtime), `backoffice`, `mobile-app`.
- **Repositorios de soporte**: `secrets`, `platform-infra`,
  `observability-platform`, `contracts-and-schemas`, `shared-sdk-client`,
  `docs`.
- **Distribuciones**: `dev` (PC del developer + VPS dev), `stg` (VPS),
  `pro` (VPS + ai-engine local).
- **Supuestos de confianza**:
  - El proveedor de identidad (Firebase Auth) es de confianza.
  - El registry de imágenes (GHCR) está protegido por token rotativo.
  - GitHub branch protection y `security.yml` (CodeQL + Trivy) están
    activos en los 8 repos runtime.
  - El operador con acceso al PC que despliega `ai-engine` es interno.

## Trust boundaries

```
[ Client mobile / browser backoffice ]
              | (HTTPS, JWT)
              v
        [ api-gateway :7005 ]   <-- public boundary
              |  (HTTPS + INTERNAL_API_TOKEN)
        +-----+---------------------+
        v                           v
  [ bff-mobile ]              [ bff-backoffice ]
        |                           |
        +------+--------+-----------+
               v        v
   [ microservice-quizz, -wordpass, -users ]   [ ai-engine-api ]
                                                       |
                                                       v
                                              [ llama runtime ]
```

Boundaries críticos:

1. Internet ↔ `api-gateway` (autenticación + cuotas).
2. `api-gateway` ↔ BFFs (token interno + cabeceras de origen).
3. BFFs ↔ microservicios de dominio (token compartido por distribución).
4. `ai-engine-api` ↔ llama runtime (canal local, no expuesto a internet).
5. Cualquier servicio ↔ `secrets` (vía artefactos de despliegue, no en
   runtime).

## Activos

| Activo | Clasificación | Notas |
|---|---|---|
| Datos de usuario (`microservice-users`) | PII | Necesita cifrado en tránsito y en reposo |
| Tokens internos (`API_TOKEN`, `AI_ENGINE_*`, `INTERNAL_API_TOKEN`) | Confidencial | Rotación reglada, ver `secrets-rotation-policy.md` |
| Modelos LLM cargados localmente | Confidencial | Riesgo de exfiltración por endpoints internos |
| Prompts, datasets RAG y eval | Interno | Pueden contener IP del producto |
| Logs y métricas Prometheus | Interno | Pueden filtrar PII si no se sanea |
| Configuración runtime persistida (rutas, presets) | Interno | Cambia comportamiento sin reinicio |

## Resumen STRIDE por componente

Las celdas marcadas con `—` indican amenaza de baja relevancia o cubierta
por la mitigación de otro componente.

### `api-gateway`

| Amenaza STRIDE | Riesgo | Vector | Mitigación actual | Acciones pendientes |
|---|---|---|---|---|
| Spoofing | High | Token JWT robado o reutilizado | Validación de firma, `Authorization: Bearer`, expiración corta | Revisar TTL y refresh policy (`ai-prompting-rules.md` no aplica) |
| Tampering | Medium | Modificación de payload en tránsito | HTTPS terminado en gateway/edge | Forzar HSTS y revisar mTLS interno hacia BFFs |
| Repudiation | Medium | Falta de auditoría sobre cambios admin | `X-Correlation-ID` propagado, log estructurado | Persistir logs de admin en backend dedicado con retención >= 90 días |
| Information disclosure | Medium | CORS permisivo o errores con stack | Política CORS restringida por distribución | Auditoría trimestral CORS + revisar mensajes de error |
| Denial of service | High | Saturación pública | Timeouts duros documentados (`inter-service-communication.md`) | Añadir rate limiting por IP + circuit breaker hacia BFFs |
| Elevation of privilege | High | Cabecera `X-Internal-Token` filtrada | Token rotado por `secrets/scripts/bootstrap-secrets.mjs` | Mover a token firmado por distribución y validar en BFFs |

### `bff-mobile` y `bff-backoffice`

| Amenaza STRIDE | Riesgo | Vector | Mitigación actual | Acciones pendientes |
|---|---|---|---|---|
| Spoofing | High | Llamadas directas saltándose el gateway | `INTERNAL_API_TOKEN` exigido en cada request | Bloquear binding público en VPS (solo overlay interno) |
| Tampering | Medium | Manipulación del payload reenviado | Validación de schema en BFF antes de upstream | Generar contratos desde `contracts-and-schemas/` y fallar en CI ante drift |
| Repudiation | Medium | Acciones admin (backoffice) sin trazabilidad | Logs con `correlation_id` + role del operador | Añadir audit trail persistente para acciones críticas (cambios de runtime, rotaciones) |
| Information disclosure | Medium | Filtrado de metadatos internos al cliente | Respuestas tipadas y sanitizadas | Añadir test de contrato negativo (no propiedades internas) |
| Denial of service | Medium | Llamadas concurrentes a `ai-engine` | Timeouts (8s dominio / 20s ai-engine) | Cola/limit por usuario y backpressure declarado |
| Elevation of privilege | High (backoffice) | Bypass de `roleHasBackofficeAccess` | Validación en login + role server-side | Añadir test E2E que verifique 403 a rutas admin sin rol |

### Microservicios de dominio (`quizz`, `word-pass`, `users`)

| Amenaza STRIDE | Riesgo | Vector | Mitigación actual | Acciones pendientes |
|---|---|---|---|---|
| Spoofing | Medium | Llamadas con token de otro servicio | `API_TOKEN` por distribución | Pasar a tokens por servicio cuando madure el modelo |
| Tampering | Medium | Inyección en parámetros de query | Validación con Zod / Pydantic | Auditar endpoints sin validador y cubrirlos en tests |
| Repudiation | Medium | Cambios sobre datos de usuario sin trazabilidad | Logs estructurados | Añadir tabla `audit_log` para mutaciones en `users` |
| Information disclosure | High (`users`) | Endpoints que devuelven más PII de la necesaria | Schemas `Public*` separados | Revisión periódica + Trivy fs / SAST cubre el resto |
| Denial of service | Medium | Bucket sin limit en endpoints públicos vía gateway | Limit aplicado en gateway (planificado) | Definir SLO por endpoint y alertar en `observability-platform` |
| Elevation of privilege | Medium | Control de roles solo en BFF | Validación duplicada server-side parcial | Mover decisiones de autorización al microservicio dueño del recurso |

### `ai-engine` (api + stats + llama runtime)

| Amenaza STRIDE | Riesgo | Vector | Mitigación actual | Acciones pendientes |
|---|---|---|---|---|
| Spoofing | High | Reuse de `AI_ENGINE_API_KEY` | Token por servicio + rotación 90d | Firmar requests con `correlation_id + nonce` (backlog) |
| Tampering | High | Prompt injection / data poisoning RAG | Reglas en [`ai-prompting-rules.md`](../guides/ai-prompting-rules.md), sanitización de input, schema validation | Añadir clasificador de jailbreak ligero antes del LLM |
| Repudiation | Medium | Cambios de prompt sin trazabilidad | `PROMPT_VERSIONS` + commits versionados | Persistir `prompt_version` en cada respuesta y exponerlo en métricas |
| Information disclosure | High | Filtrado del system prompt o de docs RAG | System prompt prohibe revelar instrucciones | Test de regresión que envía prompts de exfiltración conocidos |
| Denial of service | High | Prompts gigantes / loops de generación | Tamaños máximos, timeouts y `llm_fallback_total` | Alerta Prometheus si `rate(ai_engine_llm_fallback_total) > 0.1 / 5m` |
| Elevation of privilege | Medium | Endpoints admin (`/admin/*`) accesibles sin token | Token específico (`AI_ENGINE_BRIDGE_API_KEY`) | Separar binding del puerto admin a interfaz interna |

### `backoffice` y `mobile-app`

| Amenaza STRIDE | Riesgo | Vector | Mitigación actual | Acciones pendientes |
|---|---|---|---|---|
| Spoofing | Medium | Suplantación de sesión Firebase | Firebase Auth + role server-side en BFF | Forzar reauth ante cambios sensibles |
| Tampering | Medium | Manipulación del bundle servido | Tags inmutables (ADR 0007) + integridad del registry | Habilitar SRI en assets cuando aplique |
| Repudiation | Low | Acciones del usuario sin log | `correlation_id` enviado en cada request | — |
| Information disclosure | Medium | Logs de cliente con PII | Logging mínimo en producción | Auditar `console.log` antes de release |
| Denial of service | Low | Cliente puede saturar gateway | Limites en gateway | — |
| Elevation of privilege | Medium | Manipulación del token en localStorage | Token en memoria + refresh corto | Mover a cookies HttpOnly cuando se evalúe |

### Soporte: `secrets`, `platform-infra`, `observability-platform`

| Amenaza STRIDE | Riesgo | Vector | Mitigación actual | Acciones pendientes |
|---|---|---|---|---|
| Spoofing | Medium | Workflow malicioso suplantando dispatch | `PLATFORM_INFRA_DISPATCH_TOKEN` rotado | Limitar `permissions:` por workflow y exigir `environments` con reviewers |
| Tampering | High | Commit con secreto hardcodeado | `audit-hardcoded-secrets.mjs` + push protection | Añadir pre-commit hook obligatorio para todos los repos |
| Repudiation | Medium | Cambios de infra sin owner claro | CODEOWNERS por carpeta | Mantener vigente y revisar en retros |
| Information disclosure | High | Logs de pipelines exponen secretos | `::add-mask::` y revisión de outputs | Documentar en `docs/operations/cicd-workflow-map.md` qué outputs son seguros |
| Denial of service | Low | Workflow infinito | Timeouts por job (default GitHub) | — |
| Elevation of privilege | High | Token con `repo` scope filtrado | Tokens por uso (dispatch / read) | Migrar a fine-grained tokens y validar `permissions:` mínimo |

## Riesgos top y siguiente iteración

1. **Prompt injection y exfiltración en `ai-engine`** (Tampering / Disclosure
   high). Backlog: clasificador de jailbreak + suite de prompts de regresión.
2. **DoS en `api-gateway`** (DoS high). Backlog: rate limit por IP y circuit
   breaker hacia BFFs.
3. **Audit trail de acciones admin** (Repudiation medium en BFFs y soporte).
   Backlog: persistencia centralizada con retención >= 90 días.
4. **Rotación y scope mínimo de tokens CI/CD** (Spoofing / EoP medium-high).
   Backlog: migrar a fine-grained tokens y exigir `environments` con
   reviewers.

## Cadencia y mantenimiento

- Revisión trimestral en la retro de arquitectura
  ([`retrospective-process.md`](./retrospective-process.md)).
- Actualización ad-hoc cuando se incorpore un servicio nuevo o cambie un
  trust boundary (publicar nuevo endpoint, conectar proveedor externo, etc.).
- Cualquier acción pendiente no resuelta a los 90 días se eleva a riesgo
  aceptado o se replanifica en la retro siguiente.
