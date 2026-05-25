# Lambda — Errores Comunes en Examen

Notas generadas durante práctica de preguntas tipo examen.
Cada nota analiza UN error específico y referencia el apunte largo para profundizar.

---

## Error #1: Reintentos async y DLQ

**Pregunta**: Una función Lambda es invocada por S3 y falla. ¿Qué sucede?

**Mi error**: Dije que "Lambda intenta 3 veces y lo manda a una DLQ de S3"

**Qué estoy confundiendo**:
1. **No son 3 intentos, son 2 reintentos** → ejecución original + 2 reintentos = 3 ejecuciones totales. Se dice "2 retries"
2. **La DLQ no es "de S3"** → se configura en **Lambda**, no en S3
3. **La DLQ NO es automática** → hay que configurarla. Sin DLQ, después de los 2 reintentos **el evento se pierde**

**El flujo real**:
```
S3 evento → Lambda (falla) → reintento 1 (falla) → reintento 2 (falla) → DLQ (SOLO si la configuraste)
```

**Clave para examen**: Si preguntan "¿qué pasa sin DLQ configurada?" → el evento SE PIERDE.

**Profundizar en**: [05-lambda/03-async-invocation.md](../05-lambda/03-async-invocation.md)

---

## Error #2: DynamoDB Streams — Lambda se bloquea al fallar

**Pregunta**: Tenés DynamoDB con Streams habilitado y Lambda como consumer. Un item se actualiza pero Lambda falla al procesar. ¿Qué pasa?

**Mi error**: No supe responder — no tengo claro cómo funciona DynamoDB Streams con Lambda.

**Lo que hay que saber**:
1. DynamoDB Streams usa **Event Source Mapping** (polling, síncrono) — NO es invocación asíncrona
2. El procesamiento es **secuencial por shard (partición)**
3. Si Lambda falla, **reintenta el MISMO batch una y otra vez** — NO avanza al siguiente registro
4. Se queda bloqueado hasta que: el registro expire (24h), se procese bien, o configures un destino de fallo

**¿Por qué es peligroso?** Si Lambda sigue fallando, **BLOQUEA todo el shard**. Ningún registro nuevo de esa partición se procesa.

**Opciones para desbloquear** (esto cae en examen):
- `maxRetryAttempts` — limitar reintentos
- `maxRecordAge` — descartar registros viejos
- `on-failure destination` — mandar a SQS/SNS y seguir
- `bisectBatchOnFunctionError` — partir el batch a la mitad para aislar el registro malo

**Clave para examen**: Event Source Mapping (Streams, SQS, Kinesis) = se bloquea al fallar. Invocación asíncrona (S3, SNS) = reintenta 2 veces y descarta/DLQ.

**Profundizar en**: `05-lambda/16-concurrency-and-throttling.md` (parcial) + apuntes de DynamoDB cuando se cubran

---

## Error #3: Credenciales en Lambda — env vars con KMS vs Secrets Manager/SSM

**Pregunta**: ¿Forma RECOMENDADA de manejar credenciales de RDS en Lambda?

**Mi error**: Elegí variables de entorno con KMS. Funciona, pero NO es lo recomendado.

**Regla para examen**: "credenciales" + "recomendado" = **Secrets Manager o SSM Parameter Store (SecureString)**. Env vars con KMS es para config no-sensible.

**¿Por qué?** Secrets Manager/SSM dan rotación automática, un solo lugar para actualizar, y las credenciales no quedan visibles en la consola de Lambda.

**Profundizar en**: `05-lambda/24-best-practices.md`

---

## Error #4: Task timed out — solución directa vs re-arquitectura

**Pregunta**: Lambda con timeout default (3s) falla con archivos grandes. ¿Qué hacer?

**Mi error**: Elegí dividir archivos (re-arquitectura) en vez de aumentar el timeout.

**Regla para examen**: si el error dice "timed out" y el timeout está en default (3s) → **aumentar el timeout** (max 15 min). Re-arquitectura (dividir, Step Functions, ECS) solo si ya estás en 15 min y sigue fallando.

**Clave**: el examen pide la solución MÁS DIRECTA al problema descrito, no la más elegante.

**Profundizar en**: `05-lambda/01-fundamentals.md`

---

## Error #5: Execution Role vs Resource-Based Policy — los tenía al revés

**Pregunta**: Lambda necesita leer SQS y escribir DynamoDB. ¿Dónde configurás los permisos?

**Mi error**: Elegí Resource-Based Policy pensando que "resource-based = acceder a recursos".

**La regla**:
- **Execution Role** = qué puede HACER Lambda hacia afuera (leer SQS, escribir DynamoDB, loggear)
- **Resource-Based Policy** = quién puede LLAMAR a Lambda (S3, API Gateway, otra cuenta)

**Para examen**: Lambda necesita hacer algo → Execution Role. Algo necesita invocar Lambda → Resource-Based Policy.

**Profundizar en**: `05-lambda/08-execution-role-vs-resource-policy.md`

---

## Error #6: SQS + Lambda batch — todo o nada

**Pregunta**: Lambda procesa batch de 10 mensajes de SQS y falla en el #7. ¿Qué pasa?

**Mi error**: Pensé que Lambda reintentaba solo el mensaje fallido internamente.

**La realidad**: Los 10 mensajes vuelven a la cola (después del visibility timeout). Lambda no sabe cuáles procesaste bien — falló y punto. Los mensajes 1-6 se procesan DOS veces → tu código DEBE ser idempotente.

**Solución recomendada**: `ReportBatchItemFailures` en el Event Source Mapping. Tu función retorna la lista de mensajes fallidos → solo ESOS vuelven a la cola.

**Para examen**: SQS batch + fallo = TODO vuelve. `ReportBatchItemFailures` = solo fallos vuelven.

**Profundizar en**: `05-lambda/16-concurrency-and-throttling.md` + documentación SQS Event Source Mapping

---

## Error #7: CloudFormation no detecta nuevo ZIP en S3

**Pregunta**: Subís nuevo código Lambda a S3, hacés update-stack, pero sigue el código viejo. ¿Por qué?

**Mi error**: No estaba seguro del por qué (respondí bien por instinto).

**La trampa**: CloudFormation compara el TEMPLATE, no el contenido del archivo en S3. Si el S3Key no cambió en el template → CloudFormation no detecta cambio → no actualiza.

**Solución**: Cambiar el S3Key cada vez (`lambda-v2.zip`, `lambda-abc123.zip`). SAM lo hace automáticamente con `sam package`.

**Profundizar en**: `05-lambda/17-cloudformation.md`

---

## Error #8: Kinesis/Streams — concurrencia por shard, no global

**Pregunta**: Lambda consume Kinesis con 4 shards. ¿Máximo de invocaciones concurrentes por defecto?

**Mi error**: Dije 1 (recordaba "1 por shard" pero olvidé multiplicar por el número de shards).

**La regla**: Streams (Kinesis, DynamoDB Streams) = **1 invocación concurrente POR SHARD**. Total = número de shards. Con 4 shards → 4 concurrentes.

**Extra**: `parallelizationFactor` (hasta 10) permite más de 1 por shard, pero default es 1.

**Profundizar en**: `05-lambda/16-concurrency-and-throttling.md` + apuntes Kinesis cuando se cubran

---

## Error #9: Timeout 15 min es HARD LIMIT — no se puede subir

**Pregunta**: Lambda ya está en 15 min (máximo) y sigue fallando por timeout. ¿Qué hacer?

**Mi error**: Pensé que se podía subir contactando AWS Support.

**La regla completa**:
- Timeout en default (3s) → **aumentar timeout**
- Timeout ya en 15 min y no alcanza → **cambiar de servicio** (ECS/Fargate/Step Functions)
- 15 min es hard limit. NO se sube. No hay excepción.

**Profundizar en**: `05-lambda/23-limits-and-deployment.md`

---

## Error #10: CloudFormation + S3 Versioning — S3ObjectVersion

**Pregunta**: Template usa S3Bucket, S3Key y S3ObjectVersion. Subís código nuevo al mismo key. ¿Qué actualizar?

**Mi error**: Elegí S3Key, pero el bucket tiene Versioning activado y el key no cambió.

**La regla completa**:
- Bucket SIN versioning → cambiar `S3Key` (nombre del archivo)
- Bucket CON versioning → cambiar `S3ObjectVersion` (version ID)

Ambas logran lo mismo: que CloudFormation detecte un cambio en el template y actualice Lambda.

**Profundizar en**: `05-lambda/17-cloudformation.md`

---

## Error #11: Event Source Mapping vs Invocación asíncrona — quién usa qué

**Pregunta**: ¿Cuál NO requiere Event Source Mapping? → SNS (es asíncrono, push directo).

**Mi error**: No tenía claro qué servicios usan Event Source Mapping y cuáles no.

**La regla**:
- **Event Source Mapping (Lambda hace polling)**: SQS, Kinesis Data Streams, DynamoDB Streams → Lambda VA A BUSCAR los datos
- **Invocación asíncrona (push directo)**: SNS, S3, EventBridge → el servicio LE MANDA los datos a Lambda

**Truco**: streams y colas = polling (ESM). Notificaciones y eventos = push (asíncrono).

**Profundizar en**: `05-lambda/03-async-invocation.md`

---

## Nota #1: CloudWatch vs X-Ray vs CodeGuru Profiler

- **CloudWatch** = ¿QUÉ pasó? (logs, métricas, alarmas)
- **X-Ray** = ¿POR DÓNDE pasó y dónde se tardó? (tracing distribuido entre servicios)
- **CodeGuru Profiler** = ¿QUÉ DENTRO de mi código es lento? (profiling interno)

**Para examen**: monitorear errores → CloudWatch. Debuggear latencia entre servicios → X-Ray. Performance interno → CodeGuru.

**Profundizar en**: `05-lambda/09-logs-metrics-xray.md` y `05-lambda/22-codeguru-profiler.md`

---

## Error #12: Variables de entorno Lambda — límite 4 KB total

**Pregunta**: Token de 8 KB para API externa. ¿Dónde ponerlo?

**Clave**: Env vars tienen límite de **4 KB total** (todas combinadas). Si supera 4 KB → no cabe. Alternativas: dentro del `.zip`, S3, SSM Parameter Store, o Secrets Manager.

**Profundizar en**: `05-lambda/23-limits-and-deployment.md`

---

## Error #13: Lambda en CloudFormation — solo dos formas

**Pregunta**: ¿Cómo se declara Lambda en CloudFormation?

**Respuesta**: `.zip` en S3 referenciado en `AWS::Lambda::Function` (o inline con `ZipFile` para scripts cortos). NO existe referencia a CodeCommit, CodeDeploy, ni carpetas sueltas.

**Profundizar en**: `05-lambda/17-cloudformation.md`

---
