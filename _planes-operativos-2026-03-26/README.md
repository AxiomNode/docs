# Planes Operativos Personalizados (Temporal)

Este paquete contiene planes de mejora para:

1. Cada servicio.
2. Cada repositorio.
3. La comunicacion entre servicios.

Objetivo global:

- Que cada componente cumpla su responsabilidad principal.
- Que todo sea medible y observable de extremo a extremo.
- Que el sistema sea eficaz (cumple objetivos), eficiente (usa bien recursos) y util (entrega valor).

Alcance cubierto:

- Servicios: backoffice, api-gateway, bff-backoffice, bff-mobile, microservice-users, microservice-quizz, microservice-wordpass, ai-engine-api, ai-engine-stats, llama-server.
- Repositorios: todos los de la raiz de AxiomNode con foco operativo.

Documentos:

- `02-planes-por-repositorio.md`
- `03-plan-comunicacion-interservicios.md`
- `05-informe-resultados-iteracion-01.md` (historico de ejecucion cerrada)

Notas de uso:

- Carpeta temporal para trabajo operativo, pensada para borrado posterior.
- Los planes incluyen prioridad (`P0`, `P1`, `P2`), KPI objetivo, y criterio de exito verificable.
- Recomendacion: ejecutar por iteraciones quincenales con demo de resultados y dashboard unico.
- 2026-03-28: se eliminaron los planes/comprobaciones ya completados (`01` y `04`) para mantener solo frentes activos.

## Checklist de seguimiento

- [x] Checklist global creada por plan (servicios, repos, comunicacion).
- [x] Responsables asignados por bloque P0.
- [x] KPI baseline capturado antes de cambios.
- [x] Evidencia de despliegue y verificacion adjunta por cada item completado.
