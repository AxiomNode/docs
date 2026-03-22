# Environments And Secrets

## Configuracion por entorno

- Variables no sensibles en manifiestos/versionado.
- Secretos en gestor dedicado (por ejemplo, cloud secret manager).
- Nombres y claves consistentes entre servicios.

## Politica de secretos

- Rotacion periodica.
- No commitear secretos en repositorio.
- Acceso por roles de servicio.

## Checklist minimo por servicio

- `SERVICE_NAME`
- `SERVICE_PORT`
- `ALLOWED_ORIGINS`
- Credenciales de DB/cache
- Claves de proveedores externos
