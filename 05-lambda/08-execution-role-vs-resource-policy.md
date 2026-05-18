# Lambda — Rol de ejecución vs política basada en recursos

Resumen corto para no confundir permisos en examen.

---

## Rol de ejecución de Lambda (Lambda Execution Role)

Es un rol de **AWS Identity and Access Management (IAM)** que usa la función Lambda para acceder a recursos de AWS.

Ejemplos:

- Escribir logs en **Amazon CloudWatch Logs**
- Leer/escribir en **Amazon DynamoDB**
- Publicar en **Amazon Simple Notification Service (Amazon SNS)**

En **Mapeo de fuente de eventos (Event Source Mapping, ESM)**, Lambda usa este rol para leer la fuente (por ejemplo **Amazon Simple Queue Service (Amazon SQS)**, **Amazon Kinesis Data Streams**, **Amazon DynamoDB Streams**).

---

## Política basada en recursos (Resource-Based Policy)

Es una política adjunta a la función Lambda para permitir que **otros** servicios/cuentas/principales la invoquen.

Ejemplo típico:

- **Amazon Simple Storage Service (Amazon S3)** autorizado a ejecutar `lambda:InvokeFunction`.

---

## Regla mental

- **Execution Role** = permisos de salida de la función (lo que la función puede hacer)
- **Resource-Based Policy** = permisos de entrada (quién puede invocar la función)

---

## Buena práctica

- Un rol de ejecución por función.
- Aplicar principio de mínimo privilegio.
