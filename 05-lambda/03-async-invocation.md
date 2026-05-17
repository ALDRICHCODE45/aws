# Lambda — Invocación asíncrona

Cuando un servicio AWS empuja eventos a Lambda sin esperar respuesta.

---

## Flujo

```
S3 / SNS / EventBridge
   ↓ (push, InvocationType=Event)
Cola interna de Lambda (gestionada por AWS, invisible)
   ↓
Lambda Function
   ↓ (si falla)
Reintentos automáticos
   ↓ (si sigue fallando)
DLQ o Destinations
```

> ⚠️ La "cola de eventos" es **interna de Lambda**. NO la creás ni la ves. Distinta de SQS propia (eso es Event Source Mapping).

---

## Asíncrono vs Event Source Mapping

| Aspecto | Asíncrono | Event Source Mapping |
|---|---|---|
| Servicios típicos | S3, SNS, EventBridge | SQS, Kinesis, DynamoDB Streams |
| Quién empuja | Servicio **PUSH** a Lambda | Lambda **POLL** a la fuente |
| Cola | Interna invisible | Vos la creás / ya existe |
| Respuesta al caller | 202 Accepted | (no aplica) |

**Regla**: servicio PUSHEA → async. Lambda LEE cola → ESM.

---

## Reintentos

| Setting | Default | Rango |
|---|---|---|
| `MaximumRetryAttempts` | **2 reintentos (3 intentos totales)** | 0 - 2 |
| `MaximumEventAgeInSeconds` | **6 horas** | 60 s - 6 h |
| Backoff | ~1 min, ~2 min | Fijo |

⚠️ Reintentos = **logs duplicados en CloudWatch** (esperado).

Si fallan todos los intentos:
- Con **DLQ/Destinations** → evento va ahí
- Sin nada → **descartado silenciosamente**

---

## DLQ vs Destinations

| Feature | Captura | Destinos |
|---|---|---|
| **DLQ** (viejo) | Solo fallos | SQS, SNS |
| **Destinations** (recomendado) | OnSuccess + OnFailure | SQS, SNS, Lambda, EventBridge |

> 🎯 Examen: **Destinations** es la respuesta moderna recomendada.

Ambos requieren permisos IAM en el execution role.

---

## IDEMPOTENCIA (lo crítico)

> Función idempotente = ejecutarla N veces con el mismo input produce el mismo resultado que 1 vez.

### Por qué importa
Lambda garantiza **at-least-once**, NO exactly-once. Un evento puede procesarse más de una vez por:
- Reintentos automáticos
- Duplicados raros del sistema distribuido
- Errores transitorios invisibles

### Qué pasa si NO sos idempotente
Cobros duplicados, emails repetidos, filas duplicadas en DB.

### Patrones
1. **Idempotency key**: usar `event.id` como key en DynamoDB
2. **Conditional writes**: `attribute_not_exists(eventId)`
3. **Operaciones idempotentes**: `UPDATE`/`PUT`, no `INSERT`
4. **AWS Powertools**: decorador `@idempotent`

```javascript
exports.handler = async (event) => {
  const eventId = event.Records[0].responseElements["x-amz-request-id"];
  if (await alreadyProcessed(eventId)) return;
  await doWork();
  await markAsProcessed(eventId);
};
```

---

## Servicios que invocan asíncronamente

S3, SNS, **EventBridge**, CloudWatch Logs, SES, CodeCommit, IoT Core.

---

## Checklist examen

- [ ] Caller recibe **HTTP 202** inmediato
- [ ] Cola es **interna de Lambda**, invisible
- [ ] **At-least-once** → **idempotencia obligatoria**
- [ ] Reintentos default: **2 (3 totales)**, backoff 1min/2min
- [ ] Max event age: **6 h** (configurable)
- [ ] **Destinations** > DLQ (más capaz)
- [ ] DLQ = SQS o SNS, solo fallos
- [ ] Destinations = SQS, SNS, Lambda, EventBridge (success + failure)
- [ ] Servicios PUSH async: **S3, SNS, EventBridge**
- [ ] Diferencia ESM: ESM = Lambda pollea (SQS, Kinesis, DynamoDB Streams)
