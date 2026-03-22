# Deployment Strategy

## Entornos

- Dev: rapido, costo optimizado, datos no productivos.
- Stg: espejo operacional de produccion para validaciones.
- Prod: alta disponibilidad y controles de cambio.

## Pipeline recomendado

1. Pull Request: lint, tests, build, contract checks.
2. Merge a main: build de imagen y escaneo.
3. Deploy a dev automatico.
4. Promocion a stg tras checks de humo.
5. Promocion a prod con aprobacion.

## Estrategia de versionado

- SemVer para servicios y SDKs.
- Versionado explicito de contratos (`v1`, `v2`).
- Deprecacion con ventana definida y comunicacion previa.
