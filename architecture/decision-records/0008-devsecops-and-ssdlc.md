# ADR 0008 - DevSecOps y SSDLC como base de seguridad de la plataforma

- Status: Accepted
- Date: 2026-04-22

## Context

El bloque 9 del *Máster en Desarrollo con IA* exige aplicar *security by design* y *security by default*, OWASP Top 10 2025, SSDLC y DevSecOps con shift-left. La plataforma AxiomNode hoy gestiona secretos de forma centralizada, aplica pod security baseline en Kubernetes y ejecuta `npm audit` puntual, pero no tiene:

- Un threat model formal por servicio.
- SAST y SCA integrados en CI de forma sistemática.
- Políticas explícitas frente a prompt injection en `ai-engine` y BFFs.
- Rotación documentada de secretos.

Sin un marco SSDLC explícito, la seguridad seguirá siendo reactiva y dependerá del criterio puntual de quien revise cada PR.

## Decision

Adoptar SSDLC + DevSecOps como base operativa de seguridad de la plataforma, con los siguientes compromisos vinculantes:

1. **Análisis estático y de dependencias en CI**: cada repo de runtime ejecutará en su pipeline al menos un escáner SCA (Snyk, Trivy o `npm audit --audit-level=high` / `pip-audit`) y un escáner SAST (CodeQL o Semgrep). Los hallazgos *high* y *critical* fallan el pipeline; los *medium* abren issue automático.
2. **Threat modeling por servicio**: cada microservicio y BFF redactará un threat model ligero (STRIDE) en su propio repo bajo `docs/security/threat-model.md` antes de su próximo cambio funcional relevante. Plantilla a publicar en `docs/operations/threat-model.md`.
3. **OWASP Top 10 2025 como checklist obligatoria** en revisiones de PR que toquen endpoints públicos (`api-gateway`), BFFs o autenticación.
4. **Endurecimiento contra prompt injection** en `ai-engine` y en cualquier BFF que enrute a `ai-engine`: validación de entrada, separación de instrucciones de sistema y datos de usuario, listas de bloqueo y truncado defensivo. Documentado como práctica obligatoria; auditable por code review.
5. **Rotación de secretos**: documentar en `secrets/` la cadencia mínima (al menos anual para credenciales largas, inmediata ante incidente) y dejar registro en `secrets/scripts/`.

## Alternatives considered

1. Mantener seguridad ad-hoc por revisión humana: rechazado; no escala y no aplica shift-left.
2. Integrar sólo SCA y dejar SAST fuera: rechazado; la mayor parte de los hallazgos relevantes en backoffice y BFFs vendrían de SAST.
3. Externalizar todo a un servicio gestionado (por ejemplo Snyk Pro): viable, pero la decisión de proveedor se difiere; el ADR fija el *qué*, no el *quién*.

## Consequences

- Pipelines de CI más lentos (segundos a minutos extra por escáner). Aceptable.
- Backlog inicial elevado al ejecutar el primer pase de threat modeling. Se distribuye por repo.
- Mayor confianza objetiva en cumplimiento OWASP y en la postura ante prompt injection.
- Necesidad de un secreto adicional `SNYK_TOKEN` (o equivalente) en repos. Se gestionará junto a `CODECOV_TOKEN`.

## Related

- [Master alignment](../master-alignment.md) - pilar 9 Seguridad
- [Quality attribute scenarios](../quality-attributes.md) - escenarios de seguridad
- [Test coverage policy](../../operations/test-coverage-policy.md)
