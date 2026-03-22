# Service Communication Model

## Flujo sincrono

1. Cliente mobile o backoffice llama a `api-gateway`.
2. `api-gateway` valida politicas de borde (auth, rate limit, CORS, headers).
3. `api-gateway` enruta al BFF correspondiente.
4. BFF agrega y transforma respuestas consultando microservicios internos.

## Flujo asincrono

- Procesos de larga duracion y desacople entre dominios se manejan por eventos.
- Los contratos de eventos se versionan en `contracts-and-schemas`.

## Reglas de comunicacion

- No exponer microservicios de dominio directamente a internet.
- Evitar llamadas BFF a BFF.
- Definir timeouts y retries por dependencia.
- Incluir correlation-id en toda peticion.
