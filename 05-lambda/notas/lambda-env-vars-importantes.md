# Lambda — variables de entorno importantes

## X-Ray tracing

| Variable | Qué hace |
| -------- | -------- |
| **`_X_AMZN_TRACE_ID`** | Trace ID actual (Lambda lo inyecta auto) |
| **`AWS_XRAY_CONTEXT_MISSING`** | Comportamiento si falta contexto (`LOG_ERROR`, `RUNTIME_ERROR`, `IGNORE_ERROR`) |
| `AWS_XRAY_DAEMON_ADDRESS` | Dirección del daemon (NO Lambda, sí EC2/ECS) |

## Variables del runtime (info de la ejecución)

| Variable | Para qué |
| -------- | -------- |
| `AWS_REGION` | Región |
| `AWS_LAMBDA_FUNCTION_NAME` | Nombre función |
| `AWS_LAMBDA_FUNCTION_VERSION` | Versión |
| `AWS_LAMBDA_LOG_GROUP_NAME` | Log group CloudWatch |
| `AWS_LAMBDA_LOG_STREAM_NAME` | Log stream |
| `AWS_EXECUTION_ENV` | Runtime (ej: `AWS_Lambda_nodejs20.x`) |
| `LAMBDA_TASK_ROOT` | Path del código |
| `LAMBDA_RUNTIME_DIR` | Path del runtime |

## Trampa por convención de nombres

`_X_AMZN_TRACE_ID` **NO empieza con `AWS_`** pero es la variable más importante para X-Ray.

Descartar por "no empieza con AWS_" → te perdés esa.

## Variables INVENTADAS que aparecen como distractores

- `AWS_XRAY_LOG_LEVEL` ❌
- `XRAY_TRACE_CONFIG` ❌
- `ENABLE_TRACING` ❌

Memoria > inferencia para variables AWS.
