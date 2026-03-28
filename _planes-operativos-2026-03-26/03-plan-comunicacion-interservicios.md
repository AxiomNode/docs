# 03. Plan De Comunicacion Entre Servicios (Pendientes Vigentes)

Fecha de actualizacion: 2026-03-28.

Este documento conserva solo tareas no cerradas.

## Checklist de implantacion (comunicacion)

### Contratos

- [ ] Matriz productor-consumidor publicada.
- [ ] Breaking changes bloqueadas sin version.
- [ ] Contract tests en CI de productores y consumidores.

### Seguridad de enlace

- [ ] Tabla de auth por ruta documentada y vigente.
- [ ] Rotacion de claves automatizada en entorno objetivo.
- [ ] mTLS interno para enlaces criticos.

### Resiliencia

- [ ] Retry policy separada por error transitorio/permanente.
- [ ] Dead letter/cola de compensacion para procesos asincronos.
- [ ] Chaos testing controlado en staging.

### Observabilidad

- [ ] Trazas distribuidas activas en journeys criticos.
- [ ] Dashboard por journey `backoffice -> bff-backoffice -> microservice -> ai-engine`.
- [ ] Score de salud por cadena de dependencias.

### Calidad de datos en transito

- [ ] Validacion schema en bordes de entrada/salida.
- [ ] Idempotencia en operaciones sensibles.
- [ ] Catalogo de errores estable y versionado.
- [ ] Monitoreo de drift semantico en payloads AI.
