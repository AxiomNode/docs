# Secrets Rotation Policy

Last updated: 2026-04-23.

Procedimiento operacional para rotar secretos compartidos en AxiomNode.
Complementa a [`environments-and-secrets.md`](./environments-and-secrets.md)
y a [`secrets-distribution-guide.md`](./secrets-distribution-guide.md).

## Cadencia base

| Clase de secreto | Frecuencia mínima | Disparador adicional |
|---|---|---|
| Tokens internos compartidos (`API_TOKEN`, `INTERNAL_API_TOKEN`, `AI_ENGINE_*_API_KEY`) | 90 días | sospecha de fuga, rotación de personal |
| Tokens CI/CD (`PLATFORM_INFRA_DISPATCH_TOKEN`, `CROSS_REPO_READ_TOKEN`, `SECRETS_CROSS_REPO_READ_TOKEN`) | 180 días | cambio de owner del repo, alerta GH |
| `PRIVATE_DOCS_TOKEN` | 180 días | publicación / despublicación de docs privados |
| Credenciales de proveedor externo (registry, modelos comerciales) | según TOS del proveedor o 365 días | rotación forzada por proveedor |
| Claves de firma de paquetes / artefactos | 365 días | sospecha de compromiso |

Cualquier sospecha de fuga (commit accidental, log expuesto, alerta de
secret-scanning, baja de personal con acceso) anula la cadencia y obliga a
rotación inmediata en menos de 24 h.

## Roles

- **Secrets owner** (mantenedor del repo `secrets`): ejecuta la rotación,
  publica el nuevo valor en `secrets/runtime/*.env.secrets` y notifica.
- **Service owner** (por repo runtime): valida que su servicio levanta con el
  nuevo valor y aprueba el corte en su distribución.
- **Release captain** (rotativo, ver retros): coordina cortes simultáneos
  cuando un token afecta a varios servicios.

## Procedimiento estándar

1. **Preparar**
   - Abrir issue en `secrets` con etiqueta `rotation` y la lista de servicios
     afectados.
   - Confirmar que la ventana elegida está fuera de despliegues en curso
     (revisar `docs/operations/cicd-workflow-map.md`).
2. **Generar nuevo valor**
   - Usar `secrets/scripts/bootstrap-secrets.mjs` con `--rotate <KEY>` para
     generar valor nuevo en cada distribución (`dev`, `stg`, `pro`).
   - Validar con `secrets/scripts/validate-secrets-sync.mjs` y
     `validate-no-placeholders.mjs`.
3. **Publicar a runtime**
   - Distribuir a las VPS según `secrets-distribution-guide.md`
     (rsync/ansible o método equivalente) y al PC de despliegue de `ai-engine`.
   - En K8s: actualizar el `Secret` correspondiente (
     `kubectl create secret generic ... --dry-run=client -o yaml | kubectl apply -f -`).
4. **Recargar servicios**
   - Reiniciar el servicio consumidor (rolling restart en K8s, `compose up -d`
     en VPS dev/stg). Para tokens compartidos, hacerlo en orden:
     productores → consumidores → gateway.
5. **Verificar**
   - `/healthz` y `/readyz` en verde.
   - Smoke test E2E mínimo (login + 1 endpoint protegido).
   - Métrica `*_requests_total{status="401"}` no presenta picos sostenidos.
6. **Cerrar**
   - Marcar el issue de rotación como `done` con fecha y hash del commit en
     `secrets`.
   - Anotar en la retro semanal si hubo incidencias.

## Rotación de emergencia (<24 h)

- Saltar el orden estricto: rotar primero el token comprometido en todas las
  distribuciones afectadas y aceptar ventana breve de errores 401/403 en
  servicios secundarios.
- Invalidar el valor antiguo donde sea posible (revocación en proveedor,
  borrado del Secret previo).
- Auditar logs de las últimas 72 h con
  `secrets/scripts/audit-hardcoded-secrets.mjs` y `gh code search`.
- Abrir post-mortem usando la plantilla descrita en
  [`retrospective-process.md`](./retrospective-process.md#post-mortems).

## Auditoría continua

- `audit-hardcoded-secrets.mjs` se ejecuta semanalmente en CI del repo
  `secrets` y bloquea merges con fugas detectadas.
- Workflows `security.yml` (CodeQL + Trivy) en cada repo runtime alertan de
  patrones de secreto en código y dependencias.
- GitHub secret-scanning + push protection deben permanecer habilitados en
  todos los repos.

## Referencias

- [`environments-and-secrets.md`](./environments-and-secrets.md).
- [`secrets-distribution-guide.md`](./secrets-distribution-guide.md).
- ADR 0008 — DevSecOps + SSDLC.
- OWASP ASVS V2 (autenticación) y V6 (gestión de secretos).
