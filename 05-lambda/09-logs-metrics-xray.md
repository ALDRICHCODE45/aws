# Lambda — Logs, métricas y rastreo con X-Ray

Resumen corto para examen.

---

## 1) Logs en Amazon CloudWatch Logs

- Guardan la salida de ejecución de la función (`console.log`, `print`, etc.).
- Incluyen líneas del sistema como `START`, `END`, `REPORT`.
- Requiere que el rol de ejecución de Lambda tenga permisos para escribir logs.

---

## 2) Métricas en Amazon CloudWatch Metrics

- Son métricas agregadas en el tiempo, no texto.
- Ejemplos clave: invocaciones, errores, duración, concurrencia, throttles.
- Se usan para alarmas y monitoreo operativo.

---

## 3) Rastreo distribuido con AWS X-Ray

- Muestra el recorrido de una solicitud entre servicios y latencias por tramo.
- En Lambda, activar **rastreo activo (Active Tracing)** en la configuración.
- Permiso típico en el rol: `AWSXRayDaemonWriteAccess`.

---

## Variables de entorno de X-Ray (muy de examen)

- `_X_AMZN_TRACE_ID`: cabecera de rastreo de la invocación.
- `AWS_XRAY_CONTEXT_MISSING`: manejo cuando falta contexto (en Lambda suele `LOG_ERROR`).
- `AWS_XRAY_DAEMON_ADDRESS`: dirección `IP:PUERTO` del daemon/agente de X-Ray.

---

## Regla mental

- **Logs** = detalle textual por ejecución.
- **Metrics** = números para alertas/tendencias.
- **X-Ray** = mapa de trazas y latencias end-to-end.
