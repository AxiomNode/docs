# 05. Informe De Resultados De Iteracion 01 (Servicios)

Fecha de corte: 2026-03-26

## 1) Alcance evaluado

Se evaluo la iteracion ejecutada sobre:

- `api-gateway`
- `bff-backoffice`
- `bff-mobile`
- `microservice-quiz`
- `microservice-wordpass`

Con foco en P0:

- forwarding/auth en hops criticos,
- timeout policy en edge/BFF,
- circuit breaker y smoke para flujos AI quiz/wordpass,
- dashboard de conversion `requested -> created` en backoffice.

## 2) Evidencia tecnica

### 2.1 Validacion de build/tests

- `api-gateway`: build OK, tests OK.
- `bff-backoffice`: build OK, tests OK.
- `bff-mobile`: build OK, tests OK.
- `backoffice`: build OK, tests OK.
- `microservice-quiz`: build OK, tests OK.
- `microservice-wordpass`: build OK, tests OK.

### 2.2 Snapshot runtime (health y monitor)

Health endpoints verificados en `localhost`:

- `7005` (api-gateway): OK
- `7011` (bff-backoffice): OK
- `7010` (bff-mobile): OK
- `7100` (microservice-quiz): OK
- `7101` (microservice-wordpass): OK

KPI observados en monitor (post-redeploy de quiz/wordpass + corrida final `count=3` por servicio):

- `microservice-quiz`
  - proceso final `requested=3`, `processed=3`, `created=1`, `failed=2`
  - `processes.requestedTotal=21`
  - `processes.createdTotal=1`
  - `generation.attemptsTotal=21`
  - `generation.successRatio=0.0476`
  - `outboundByOperation`: `generate=200` (1), `generate=422` (2), `generate=500` (6)
- `microservice-wordpass`
  - proceso final `requested=3`, `processed=3`, `created=1`, `failed=0`, `duplicates=2`
  - `processes.requestedTotal=21`
  - `processes.createdTotal=1`
  - `generation.attemptsTotal=21`
  - `generation.successRatio=0.0476`
  - `outboundByOperation`: `generate=200` (1), `generate=500` (6)

Hallazgo clave de calidad de payload (quiz):

- Se registraron errores `422` con detalle `Unknown game_type 'educational-game'` en respuestas del AI Engine durante la corrida final.
- Impacto: reduce conversion efectiva de quiz aun con auth ya establecida.
- Mitigacion aplicada en codigo: normalizacion defensiva de aliases de `game_type` en `ai-engine` (`educational-game -> quiz`).
- Estado tras mitigacion: persisten `422` por validacion de campos faltantes en payload del modelo (no por `game_type`), por lo que la deuda pasa a calidad estructural de salida LLM.

## 3) Comparativa KPI baseline vs post-cambio

## 3.1 Resiliencia edge/BFF

- Baseline (pre): politica de timeout no homogenea en hops edge/BFF.
- Post: timeout estandar aplicado con `UPSTREAM_TIMEOUT_MS` y mapping de expiracion a `504`.
- Resultado: mejora estructural aplicada y validada por build/tests.

## 3.2 Flujo AI quiz/wordpass

- Baseline (pre): incidentes por auth faltante en integraciones AI; fallos de generacion no acotados.
- Post (codigo): breaker + smoke + fail-fast 503 implementados.
- Post (runtime observado): desaparecen `403` en operaciones AI para quiz/wordpass tras alinear key efectiva en compose de ambos microservicios.
- Resultado: resiliencia de control implementada y autenticacion operativa recuperada; queda pendiente consolidar eficacia con nueva corrida batch.

## 3.3 Visibilidad operativa

- Baseline (pre): faltaba dashboard de conversion en overview.
- Post: dashboard `requested -> created` desplegado en backoffice.
- Resultado: mejora completada (observabilidad funcional desde UI).

## 4) Conclusiones de iteracion

- La capa edge/BFF queda endurecida (timeouts y error semantics consistentes).
- El control de dano en quiz/wordpass esta implementado y la causa raiz de `403` quedo corregida en runtime.
- La eficacia de generacion ya es medible y trazable (se obtuvo `created=1` en cada servicio en la corrida final corta), pero aun esta lejos del objetivo operativo por baja conversion global (`1/21` por servicio, ~`4.76%`).
- En quiz persiste deuda funcional en el payload del modelo (respuestas con `game_type=educational-game`) que provoca rechazos `422`.

## 5) Deuda priorizada (P1/P2)

Prioridad inmediata (operativa):

1. Endurecer post-procesado de salida LLM para quiz (campos obligatorios y estructura de `questions`) para reducir `422` remanentes.
2. Ejecutar corrida batch completa post-fix de payload y registrar nuevamente `requested -> created` por servicio.
3. Validar que `successRatio` y `failureRatio` se estabilizan en ventana operativa (sin regresion de auth).

P1 tecnico:

1. Exponer explicitamente `aiAuthCircuit` en snapshot de monitor y panel para diagnostico directo de apertura/cierre del circuito.
2. Añadir alerta de degradacion por `createdTotal/requestedTotal` por debajo de umbral.
3. Incorporar comparativa temporal (ventana ultima hora/dia) en panel de overview.

P2 tecnico:

1. Añadir reporte automatico de costo por item generado y ratio de desperdicio por intentos fallidos.
2. Integrar trazas distribuidas en todo el camino `gateway -> bff -> microservice -> ai-engine` para MTTD menor.

## 6) Decision de salida

Estado de iteracion 01: **exitosa con seguimiento**.

- Exitosa en hardening estructural, observabilidad de UI y recuperacion de autenticacion AI.
- Con seguimiento porque la conversion global sigue baja y quiz mantiene rechazos `422` por payload incompleto del modelo.

Recomendacion: abrir iteracion corta enfocada en calidad estructural de payload quiz (titulo, preguntas y campos obligatorios) y cerrar con nueva medicion de conversion sostenida post-fix.
