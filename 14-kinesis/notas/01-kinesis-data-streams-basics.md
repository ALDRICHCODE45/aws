# Kinesis Data Streams — esenciales

## Concepto base

Stream dividido en **shards** (particiones). Productores escriben con una **partition key**, Kinesis hace hash → asigna shard.

## Por shard

- Escritura: **1 MB/s o 1000 records/s**
- Lectura: **2 MB/s**
- **Orden garantizado SOLO dentro del shard**, NO entre shards.

## Misma partition key → mismo shard → orden garantizado

Para que los datos de un sensor lleguen ordenados → usar `sensor_id` como partition key.

## Disparadores

| Pregunta dice                     | Respuesta                                      |
| --------------------------------- | ---------------------------------------------- |
| "stream con N shards" + "orden"   | **Orden dentro del shard solamente**           |
| "datos del mismo X en orden"      | Usar X como **partition key**                  |
| "throughput insuficiente"         | **Aumentar shards** (resharding)               |
| "registros perdidos / reprocesar" | Aumentar **retention** (default 24h, max 365d) |

## Trampas

- "Orden exacto del stream" → **NO existe** orden global entre shards.
- "Orden inverso" → Kinesis no entrega en reverso.
- `GetRecords` no permite elegir orden, solo lee secuencialmente.

## Kinesis vs otros

| Servicio                  | Cuándo                                                      |
| ------------------------- | ----------------------------------------------------------- |
| **Kinesis Data Streams**  | Real-time con orden por key, múltiples consumidores, replay |
| **Kinesis Data Firehose** | Cargar a S3/Redshift/OpenSearch sin código                  |
| **SQS**                   | Cola 1-a-1                                                  |
| **SNS**                   | Pub/Sub fan-out                                             |
