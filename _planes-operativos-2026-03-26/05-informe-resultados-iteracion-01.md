# 05. Pendientes Vigentes Post Iteracion 01 (Servicios)

Fecha de actualizacion: 2026-03-28.

Este documento conserva solo deuda abierta. Todo bloque marcado como completado/cerrado en la iteracion fue retirado.

## 1) Prioridad inmediata (operativa)

1. Endurecer post-procesado de salida LLM para quiz (campos obligatorios y estructura de `questions`) para reducir `422` remanentes.
2. Ejecutar corrida batch completa post-fix de payload y registrar nuevamente `requested -> created` por servicio.
3. Validar que `successRatio` y `failureRatio` se estabilizan en ventana operativa (sin regresion de auth).

## 2) Pendientes P1

1. Exponer explicitamente `aiAuthCircuit` en snapshot de monitor y panel para diagnostico directo de apertura/cierre del circuito.
2. Anadir alerta de degradacion por `createdTotal/requestedTotal` por debajo de umbral.
3. Incorporar comparativa temporal (ventana ultima hora/dia) en panel de overview.

## 3) Pendientes P2

1. Anadir reporte automatico de costo por item generado y ratio de desperdicio por intentos fallidos.
2. Integrar trazas distribuidas en todo el camino `gateway -> bff -> microservice -> ai-engine` para MTTD menor.

## 4) Estado actual

- Iteracion 01 cerrada en hardening y observabilidad.
- Seguimiento activo: eficacia de generacion quiz/wordpass.
