# Retrospective Process

Last updated: 2026-04-23.

AxiomNode opera con retros ligeras pero recurrentes para mantener la
calidad de plataforma multi-repo y la alineación con el temario del máster.

## Cadencia

| Tipo | Frecuencia | Duración objetivo | Participantes |
|---|---|---|---|
| Retro semanal de plataforma | semanal (viernes) | 30 min | core team |
| Retro de release | tras cada corte a `pro` | 45 min | core team + service owners afectados |
| Post-mortem | dentro de 5 días tras un incidente Sev1/Sev2 | 60 min | responder + service owners + secrets owner si aplica |
| Retro trimestral de arquitectura | cada 3 meses | 90 min | core team + revisión cruzada |

## Retro semanal

Estructura fija (timebox sugerido entre paréntesis):

1. **Métricas clave (5 min)**
   - Estado CI por repo (verde / rojo / flaky).
   - Cobertura `ai-engine` (gate 90%).
   - Métricas LLMOps (`ai_engine_llm_fallback_total`, `errors_total`,
     `latency_p95_seconds`).
   - Alertas activas en `observability-platform`.
2. **Avances (5 min)** — qué se cerró desde la retro anterior, referenciando
   commits / PRs / ADRs.
3. **Bloqueos (5 min)** — qué impide progresar y quién lo desbloquea.
4. **Acciones (10 min)** — máximo 5 acciones, cada una con owner y fecha
   objetivo. Se trasladan al backlog o issue tracker correspondiente.
5. **Riesgos y aprendizajes (5 min)** — registrar en notas si aparece algo
   que valga la pena memorizar (ver sección "Memoria operativa").

Las notas se guardan en `docs/operations/retros/<YYYY-MM-DD>.md` (crear esta
carpeta cuando se ejecute la primera retro). Mantener una estructura mínima:
`Métricas`, `Avances`, `Bloqueos`, `Acciones`, `Riesgos`.

## Retro de release

- Revisar tags inmutables publicados (ADR 0007).
- Confirmar que `cicd-workflow-map.md` sigue alineado con la realidad.
- Validar que no quedaron secretos rotados a medias
  ([`secrets-rotation-policy.md`](./secrets-rotation-policy.md)).
- Registrar issues nuevos detectados durante el rollout.

## Post-mortems

Plantilla mínima recomendada:

```
# Post-mortem <título>
- Fecha del incidente:
- Severidad: Sev1 | Sev2 | Sev3
- Servicios afectados:
- Detección (¿quién y cuándo lo vio?):
- Línea de tiempo (UTC):
- Causa raíz:
- Impacto cuantificado:
- Acciones inmediatas:
- Acciones de prevención (con owner y fecha):
- Lecciones aprendidas:
```

Reglas:

- **Blameless**: se discuten sistemas y procesos, no personas.
- **Acciones medibles**: cada acción de prevención termina en un PR, ADR o
  cambio operativo concreto.
- **Visibilidad**: el post-mortem se guarda en
  `docs/operations/post-mortems/<YYYY-MM-DD>-<slug>.md`.

## Retro trimestral de arquitectura

- Revisar `docs/architecture/master-alignment.md` y mover brechas cerradas a
  "Avances recientes".
- Evaluar si algún ADR amerita superseed.
- Confirmar que la lista de repos en `repository-map.md` sigue completa.
- Decidir el siguiente lote de mejoras a abordar.

## Memoria operativa

Decisiones puntuales o aprendizajes con valor a largo plazo se registran en:

- ADRs (cuando hay decisión estructural).
- `docs/operations/*.md` (cuando son políticas operativas).
- Notas internas del repo `docs/next-steps/` cuando son seguimiento corto.

Si una acción de retro implica cambios en varios repos, abrir issues
coordinados y referenciarlos desde el documento de retro.

## Referencias

- [`environments-and-secrets.md`](./environments-and-secrets.md).
- [`secrets-rotation-policy.md`](./secrets-rotation-policy.md).
- [`cicd-workflow-map.md`](./cicd-workflow-map.md).
- `docs/architecture/master-alignment.md`.
