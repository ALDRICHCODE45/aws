# Lambda — Mapeo de fuente de eventos (Event Source Mapping)

Cómo **AWS Lambda** consume eventos cuando la fuente es una cola o un stream.

---

## Qué es

**Mapeo de fuente de eventos (Event Source Mapping, ESM)** es un componente interno de Lambda que:

1. Hace sondeo (polling) de la fuente.
2. Agrupa registros en lotes (batch).
3. Invoca tu función Lambda de forma **sincrónica** con ese lote.
4. Si sale bien, confirma/elimina según el tipo de fuente.
5. Si falla, aplica reintentos y reglas de error.

---

## Fuentes típicas

- **Amazon Kinesis Data Streams (Kinesis)**
- **Amazon DynamoDB Streams (DynamoDB Streams)**
- **Amazon Simple Queue Service (Amazon SQS) estándar**
- **Amazon Simple Queue Service First-In First-Out (Amazon SQS FIFO)**

---

## Diferencia clave: streams vs colas

| Tema | Streams (Kinesis / DynamoDB Streams) | Colas (Amazon SQS) |
|---|---|---|
| Lectura | Lambda lee por **fragmento (shard)** | Lambda sondea la cola |
| Orden | Se mantiene por shard | Estándar: no estricto / FIFO: sí por grupo |
| Después de procesar | Registro sigue en stream hasta retención | Mensaje se elimina si el procesamiento fue exitoso |
| Escalado | Por shard | Estándar: agresivo / FIFO: por grupos activos |

---

## Streams + Lambda (Kinesis y DynamoDB Streams)

- Se crea un iterador por shard.
- Se procesa en orden dentro de cada shard.
- Se puede iniciar en:
  - últimos registros,
  - desde el inicio,
  - o desde marca de tiempo.
- Con paralelización, pueden procesarse varios lotes en paralelo por shard (hasta 10).

> 🎯 Examen: procesar un registro NO lo borra del stream; otros consumidores pueden leerlo también.

---

## Errores en streams

Por defecto, si tu función falla en un lote:

- Lambda reintenta ese lote.
- El shard afectado puede pausarse para conservar orden.

Ajustes importantes del mapeo:

- edad máxima del registro,
- máximo de reintentos,
- dividir lote en caso de error (bisect batch on function error),
- destino para descartes (on-failure destination).

---

## Amazon SQS + Lambda

- Lambda usa **sondeo largo (long polling)** para leer mensajes.
- Procesa por lotes (batch size configurado en el trigger).
- Si todo sale bien, elimina mensajes de la cola.
- Si falla, el mensaje reaparece al terminar el **tiempo de espera de visibilidad (Visibility Timeout)**.

Regla recomendada:

> **Visibility Timeout ≈ 6 × Lambda Timeout**

Esto evita reprocesado temprano mientras la función todavía está corriendo.

---

## Amazon SQS estándar vs Amazon SQS FIFO

### Amazon SQS estándar

- No garantiza orden estricto.
- Puede haber duplicados (modelo at-least-once).
- Lambda escala para vaciar la cola lo más rápido posible.

### Amazon SQS FIFO

- Mantiene orden por **identificador de grupo de mensajes (Message Group ID)**.
- Escala según cantidad de grupos de mensajes activos.
- Mensajes del mismo grupo se procesan en orden.

---

## Dead Letter Queue (DLQ) en SQS

Si querés enviar mensajes fallidos a **cola de mensajes fallidos (Dead Letter Queue, DLQ)**:

- se configura en la **cola de Amazon SQS**,
- no en la configuración asíncrona de Lambda.

---

## Checklist examen

- [ ] Event Source Mapping (ESM) = Lambda hace polling y envía lotes a tu función
- [ ] Fuentes típicas ESM: Amazon SQS, Kinesis, DynamoDB Streams
- [ ] Streams: orden por shard, no borrado al procesar
- [ ] Streams: error en lote puede frenar shard para mantener orden
- [ ] Amazon SQS estándar: sin orden estricto, posible duplicado
- [ ] Amazon SQS FIFO: orden por Message Group ID
- [ ] Amazon SQS + Lambda: usar Visibility Timeout ≈ 6 × Lambda Timeout
- [ ] Dead Letter Queue (DLQ) para SQS se configura en la cola
