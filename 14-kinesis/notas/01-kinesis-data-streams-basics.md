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

## Patrón clave: consumidor más lento que la retención = PÉRDIDA

- Si el consumidor procesa cada **48h** pero la retención es **24h** → los registros
  EXPIRAN antes de leerse → faltan datos aguas abajo (ej: Redshift).
- Pista: choque entre el intervalo del consumidor y la retención por defecto (24h).
- Fix: subir retención (>intervalo, hasta 365d) o consumir más seguido.
- NO es culpa de Redshift (es buen destino para análisis histórico) ni de la EC2
  (si el enunciado dice que está sana, descartá esa opción).
- Opción que CONTRADICE un hecho del enunciado → descarte automático.

## Orden estricto + exactly-once (error simulacro #2)
- **`PutRecord`** (singular) soporta **`SequenceNumberForOrdering`** → orden
  estrictamente creciente para la MISMA partition key. ES la forma de ordenar.
- **`PutRecords`** (batch) NO garantiza orden en el lote + permite fallos parciales
  → rompe el orden. NO usar para orden estricto.
- **PartitionKey decide el shard. Orden solo DENTRO del shard.**
  - Para ordenar → partition key **CONSTANTE** (todas al mismo shard).
  - **timestamp como partition key = ERROR** → dispersa en shards = sin orden.
- Solución típica: `PutRecord` + ID único (exactly-once) + `SequenceNumberForOrdering`.
- Si el escenario está "sobre Kinesis", la respuesta resuelve EN Kinesis
  (no "sustituir por SQS FIFO").

## Consumidores
- **Classic**: 2 MB/s COMPARTIDO entre todos los consumidores del shard (pull).
- **Enhanced Fan-Out**: 2 MB/s POR consumidor (push, HTTP/2). Para muchos
  consumidores que necesitan throughput dedicado y baja latencia.

## Kinesis vs otros

| Servicio                  | Cuándo                                                      |
| ------------------------- | ----------------------------------------------------------- |
| **Kinesis Data Streams**  | Real-time con orden por key, múltiples consumidores, replay |
| **Kinesis Data Firehose** | Cargar a S3/Redshift/OpenSearch sin código                  |
| **SQS**                   | Cola 1-a-1                                                  |
| **SNS**                   | Pub/Sub fan-out                                             |

## Data Streams vs Firehose (la confusión #1)
- ¿Replay / reprocesar / múltiples consumidores / tiempo real custom? → **Data Streams**.
- ¿Solo entregar a S3/Redshift/OpenSearch/Splunk sin administrar? → **Firehose**
  (serverless, NO replay, near-real-time, auto-scale).

## Pregunta de prueba

Un Kinesis Data Stream recibe datos de sensores. Una EC2 consumidora procesa cada
48h y guarda en Redshift. Faltan muchos datos en Redshift; sensores y EC2 están OK.
¿Causa más probable?

A) Amazon Redshift no sirve para almacenar datos de streaming
B) Los registros se retienen 24h por defecto en el Data Stream
C) La instancia EC2 falla intermitentemente
D) Kinesis tiene demasiados shards aprovisionados

<details><summary>Respuesta</summary>

**B**: retención default 24h < intervalo de 48h → los registros expiran antes de leerse.
Cuándo sería cada una:
- **Redshift no sirve** → falso, es ideal para análisis histórico.
- **EC2 falla** → el enunciado dice que está sana (contradice el dato).
- **demasiados shards** → eso es costo/over-provisioning, no pérdida de datos.
</details>
