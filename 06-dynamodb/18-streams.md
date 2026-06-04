# DynamoDB — Streams

Stream ordenado de cambios (crear/actualizar/borrar) a nivel de ítem. Permite reaccionar en tiempo real.

---

## Qué es

- Cada cambio en la tabla genera un registro en el stream
- Retención: **24 horas**
- NO retroactivo — solo captura cambios DESPUÉS de habilitarlo (datos previos no aparecen)

## Qué manda el stream

| Opción | Contenido |
|---|---|
| `KEYS_ONLY` | Solo PK + SK |
| `NEW_IMAGE` | Ítem completo después del cambio |
| `OLD_IMAGE` | Ítem completo antes del cambio |
| `NEW_AND_OLD_IMAGES` | Ambas versiones |

## Quién puede leer el stream

- **AWS Lambda** (lo más común)
- **Kinesis Data Streams** → Kinesis Firehose → S3 / Redshift / ElasticSearch
- **KCL App** (Kinesis Client Library)

## Casos de uso

- Reaccionar en tiempo real (email de bienvenida al crear usuario)
- Insertar en tablas derivadas
- Sincronizar con ElasticSearch
- Analíticas (enviar a Redshift)
- Archivar en S3
- Replicación entre regiones

## Para el examen

- "Reaccionar a cambios en DynamoDB" → Streams + Lambda
- "Sincronizar DynamoDB con otro servicio" → Streams
- "¿Los datos existentes aparecen en el stream?" → **NO**, solo cambios nuevos
