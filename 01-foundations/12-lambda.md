# Lambda — AWS Lambda (Serverless Compute)

Apunte de consolidación con foco de examen (DVA-C02).

---

## 1) ¿Qué es Lambda?

Servicio de **compute serverless**: ejecutás código sin gestionar servidores.
AWS se encarga de aprovisionar, parchear, escalar y monitorear la infra.

### Pilares clave
- **Serverless** — vos solo subís código; AWS maneja todo lo demás.
- **Event-driven** — las funciones se ejecutan en respuesta a un evento (trigger).
- **Pay-per-use** — pagás por request + duración (redondeo a 1 ms).
- **Integración nativa** — se dispara desde casi cualquier servicio AWS (S3, DynamoDB, API Gateway, SNS, SQS, EventBridge, etc.).
- **Escalado automático** — de 0 a miles de ejecuciones concurrentes sin configurar nada.

---

## 2) Modelo de recursos (CRÍTICO para examen)

| Recurso | Rango | Detalle |
|---|---|---|
| **Memoria (RAM)** | 128 MB → 10,240 MB | En incrementos de 1 MB |
| **CPU** | No se configura directo | Escala **proporcional a la memoria** |
| **Red** | No se configura directo | Escala **proporcional a la memoria** |
| **Timeout** | 1 s → **900 s (15 min) máx** | Límite duro |
| **Ephemeral storage (/tmp)** | 512 MB → 10,240 MB | 512 MB gratis por defecto |
| **Arquitecturas** | x86_64 / **arm64 (Graviton2)** | ARM ≈ 20% más barato y hasta 34% mejor price/perf |

**Truco examen**: si una pregunta dice "necesito más CPU en mi Lambda", la respuesta es **aumentar memoria**. No hay slider de CPU.

---

## 3) Pricing

### Free tier (permanente, no se vence)
- **1,000,000 requests/mes** gratis
- **400,000 GB-segundos/mes** de duración

> ⚠️ **Atención unidad**: es **GB-segundos**, NO "GB". Se calcula:
> `memoria asignada (GB) × duración (segundos)`

### Ejemplos para entender GB-s
| Memoria | Cuántos segundos cubre el free tier |
|---|---|
| 512 MB (0.5 GB) | 800,000 s |
| 1 GB | 400,000 s |
| 2 GB | 200,000 s |
| 10 GB | 40,000 s |

### Después del free tier
- **Requests**: $0.20 por millón
- **Duración**: depende de memoria y arquitectura (x86 vs ARM)
- Lambda paga **rounded up al ms más cercano**

---

## 4) Casos de uso típicos (los que pregunta AWS)
- **Procesamiento de archivos** — trigger S3 → procesar imagen/video
- **Backends de APIs** — API Gateway → Lambda
- **Stream processing** — Kinesis / DynamoDB Streams
- **Cron jobs** — EventBridge Scheduler
- **ETL** — pipelines de datos event-driven
- **Mobile/IoT backends**

---

## 5) Checklist de examen — Sección 1

- [ ] Serverless = vos NO gestionás servidores
- [ ] Memoria: 128 MB a 10,240 MB
- [ ] Timeout máximo: **15 minutos (900 s)**
- [ ] CPU y red escalan **proporcionalmente** a la memoria
- [ ] Free tier: 1M requests + 400K **GB-s** (permanente)
- [ ] Soporta x86 y **ARM (Graviton2 — más barato)**
- [ ] Pago por request + duración (redondeo a 1 ms)

---

## 6) Invocación síncrona — Exposición HTTP/HTTPS

Lambda puede exponerse como endpoint HTTP/HTTPS de **3 formas principales**:

1. **Application Load Balancer (ALB)** — backend HTTP detrás de infra existente
2. **API Gateway** — APIs REST/HTTP/WebSocket con features avanzadas
3. **Function URL** — endpoint HTTPS directo, sin servicios intermedios

Todas son **invocación SÍNCRONA**: el cliente espera la respuesta de Lambda.

---

### 6.1) ALB + Lambda — Concepto

```
Cliente HTTP/S
     ↓
   ALB (listener :80 o :443)
     ↓
Listener Rule (¿matchea path/header/host?)
     ↓
Target Group (tipo: lambda)
     ↓
Lambda Function (1 sola por target group)
```

#### ¿Qué es un Target Group?
Es la **lista de cosas que pueden recibir tráfico** del Load Balancer.
El ALB nunca habla directo con los recursos — siempre pasa por un target group.

| Tipo de target group | Qué contiene |
|---|---|
| `instance` | EC2 instances |
| `ip` | IPs (puede ser on-premises) |
| **`lambda`** | **1 sola función Lambda** |
| `alb` | Otro ALB (NLB→ALB chaining) |

> ⚠️ **REGLA CLAVE**: un target group de tipo Lambda admite **UNA SOLA función**.
> Para exponer varias Lambdas detrás del mismo ALB → un target group por cada una + listener rules con condiciones de routing.

---

### 6.2) Cómo se comunica ALB con Lambda

| Etapa | Qué pasa |
|---|---|
| 1. Cliente → ALB | HTTP/HTTPS normal |
| 2. ALB → Lambda | ALB **convierte la request a JSON** y la pasa como `event` |
| 3. Lambda procesa | Recibe un evento con `httpMethod`, `path`, `headers`, `queryStringParameters`, `body`, `isBase64Encoded` |
| 4. Lambda → ALB | Devuelve **JSON** con `statusCode`, `headers`, `body`, `isBase64Encoded` |
| 5. ALB → Cliente | ALB **convierte el JSON a respuesta HTTP** real |

#### Event de ejemplo (request)
```json
{
  "requestContext": { "elb": { "targetGroupArn": "..." } },
  "httpMethod": "GET",
  "path": "/users",
  "queryStringParameters": { "id": "42" },
  "headers": { "host": "...", "user-agent": "..." },
  "body": "",
  "isBase64Encoded": false
}
```

#### Response esperada
```json
{
  "statusCode": 200,
  "statusDescription": "200 OK",
  "headers": { "Content-Type": "application/json" },
  "body": "{\"hello\":\"world\"}",
  "isBase64Encoded": false
}
```

> 🎯 **Headers que ALB inyecta siempre**:
> `X-Amzn-Trace-Id`, `X-Forwarded-For`, `X-Forwarded-Port`, `X-Forwarded-Proto`

---

### 6.3) Multi-value headers (OFF por defecto)

Si un header o query param se repite (ej: `?key=a&key=b` o cookies múltiples), por defecto el ALB te pasa **solo el último valor**.

| Modo | Campo en el event | Tipo de valor |
|---|---|---|
| Default (OFF) | `headers`, `queryStringParameters` | string (último valor) |
| Multi-value ON | `multiValueHeaders`, `multiValueQueryStringParameters` | array de strings |

Se habilita en los **attributes del target group** (no en el ALB ni en la Lambda).

---

### 6.4) Health checks

- Por defecto **DESHABILITADOS** para target groups tipo Lambda.
- Si los activás, Lambda recibe un evento con `user-agent: ELB-HealthChecker/2.0`.
- Útil para **DNS failover con Route 53**.
- ⚠️ Cada health check **cuenta como una invocación** (te cobran).

---

### 6.5) Límites a recordar

| Límite | Valor |
|---|---|
| Request body máx | 1 MB |
| Response máx | 1 MB |
| WebSockets | ❌ No soportados (rechaza con 400) |
| Funciones por target group | 1 |
| Lambda y target group | Misma cuenta y misma región |

---

### 6.6) ALB vs API Gateway — Decisión de examen

| Necesidad | Elegir |
|---|---|
| API REST/HTTP completa, throttling, API keys, planes de uso | **API Gateway** |
| Auth integrada con Cognito User Pools | **API Gateway** |
| **WebSockets** | **API Gateway** |
| Endpoints HTTP simples detrás de infra ya existente con ALB | **ALB** |
| Routing por path/header básico, sin features de API | **ALB** |
| Caching de respuestas | **API Gateway** |
| Request/Response transformations | **API Gateway** |

---

### 6.7) Truco examen — Tipos de ELB y Lambda

| Tipo de Load Balancer | ¿Soporta Lambda? |
|---|---|
| **ALB** (Application, Layer 7) | ✅ SÍ |
| **NLB** (Network, Layer 4) | ❌ NO |
| **GLB** (Gateway) | ❌ NO |
| **CLB** (Classic) | ❌ NO |

Si una pregunta menciona "load balancer + Lambda" → la respuesta correcta SIEMPRE es **ALB**.

---

### 6.8) Checklist de examen — Sección 6

- [ ] Lambda + Load Balancer = solo **ALB** (no NLB ni CLB)
- [ ] Target group de tipo lambda admite **1 sola función**
- [ ] Invocación **síncrona**: ALB espera respuesta
- [ ] ALB ↔ Lambda intercambia **JSON** (request y response)
- [ ] Multi-value headers están **OFF por defecto** → hay que activarlos en el target group
- [ ] Health checks están **OFF por defecto** para target groups tipo Lambda
- [ ] Lambda y target group deben estar en **misma cuenta y región**
- [ ] **No WebSockets** con ALB → usar API Gateway WebSocket
- [ ] Request/Response máx **1 MB** cada uno
- [ ] ALB inyecta headers `X-Forwarded-*` y `X-Amzn-Trace-Id`

---

## 7) Invocación asíncrona

Servicios como **S3, SNS, EventBridge (CloudWatch Events)** invocan Lambda de forma asíncrona: empujan un evento y NO esperan respuesta.

### 7.1) Flujo

```
S3 / SNS / EventBridge
       ↓ (push, InvocationType=Event)
Cola interna de Lambda (gestionada por AWS, invisible para vos)
       ↓
   Lambda Function
       ↓ (si falla)
   Reintentos automáticos
       ↓ (si sigue fallando)
   DLQ o Destinations
```

> ⚠️ La "cola de eventos" del diagrama es **interna de Lambda**. Vos NO la creás ni la ves. Es distinta de una SQS propia (eso es Event Source Mapping → otra cosa).

---

### 7.2) Asíncrono vs Event Source Mapping (CRÍTICO para examen)

| Aspecto | Invocación asíncrona | Event Source Mapping |
|---|---|---|
| **Servicios típicos** | S3, SNS, EventBridge | SQS, Kinesis, DynamoDB Streams, MSK |
| **Quién empuja** | El servicio **push** a Lambda | Lambda **pollea** activamente la fuente |
| **Cola** | Interna de Lambda (invisible) | Vos la creás / ya existe |
| **Reintentos** | 2 (3 intentos totales) por Lambda | Depende del servicio fuente |
| **InvocationType** | `Event` | (no aplica, es polling) |
| **Respuesta al caller** | 202 Accepted inmediato | (no aplica) |

**Regla mnemotécnica**: si el servicio **PUSHEA** → asíncrono. Si Lambda **LEE** una cola → event source mapping.

---

### 7.3) Reintentos y errores

| Setting | Default | Rango |
|---|---|---|
| `MaximumRetryAttempts` | 2 reintentos (3 intentos totales) | 0 - 2 |
| `MaximumEventAgeInSeconds` | 6 horas | 60 s - 6 h |
| Backoff entre reintentos | ~1 min, ~2 min | No configurable |

⚠️ Cada reintento produce **logs duplicados en CloudWatch** → no asumas que un log duplicado es un bug.

Cuando se agotan los reintentos o expira el max event age:
- Si hay **DLQ** o **Destinations** configurados → el evento va ahí.
- Si no → se **descarta silenciosamente**.

---

### 7.4) DLQ vs Lambda Destinations

| Feature | Captura | Destinos |
|---|---|---|
| **Dead Letter Queue (DLQ)** | Solo fallos | SQS, SNS |
| **Lambda Destinations** (recomendado) | **OnSuccess** + **OnFailure** por separado | SQS, SNS, Lambda, EventBridge |

> 🎯 **Examen**: si te dan opciones DLQ vs Destinations, **Destinations** es la respuesta moderna recomendada por AWS (desde 2019).

Ambas requieren **permisos IAM** correctos en el execution role de la Lambda (`sqs:SendMessage`, `sns:Publish`, etc.).

---

### 7.5) IDEMPOTENCIA (lo más importante de esta sección)

> Una función es **idempotente** si ejecutarla N veces con el mismo input produce el mismo resultado que ejecutarla 1 vez.

#### Por qué es CRÍTICO en invocación asíncrona
Lambda garantiza entrega **at-least-once**, NO exactly-once. El mismo evento puede procesarse más de una vez por:
1. **Reintentos automáticos** ante errores.
2. **Duplicados raros** del sistema distribuido.
3. **Errores transitorios** invisibles (timeouts, throttling).

#### Qué pasa si NO sos idempotente
- Cobros duplicados a clientes
- Emails repetidos
- Filas duplicadas en DB
- Inventario descontado N veces

#### Patrones para garantizar idempotencia
1. **Idempotency key**: usar `event.id` o hash del evento como clave en DynamoDB.
2. **Conditional writes**: `attribute_not_exists(eventId)` en DynamoDB.
3. **Operaciones naturalmente idempotentes**: `UPDATE` con WHERE específico, `PUT` (no `INSERT`).
4. **AWS Powertools for Lambda**: decorador `@idempotent` que maneja todo.

#### Ejemplo
```javascript
exports.handler = async (event) => {
  const eventId = event.Records[0].responseElements["x-amz-request-id"];

  if (await alreadyProcessed(eventId)) return; // ← protección

  await doWork();
  await markAsProcessed(eventId);
};
```

---

### 7.6) Servicios que invocan asíncronamente (memorizar)

- **S3** (object created, deleted, etc.)
- **SNS**
- **EventBridge** (eventos de servicios AWS + cron)
- **CloudWatch Logs** (subscription filters)
- **SES** (incoming email)
- **CodeCommit** (triggers)
- **IoT Core**

---

### 7.7) Checklist de examen — Sección 7

- [ ] Invocación asíncrona = el caller NO espera respuesta (HTTP 202)
- [ ] La cola de eventos es **interna de Lambda**, invisible
- [ ] **At-least-once** delivery → puede haber duplicados → **idempotencia obligatoria**
- [ ] Reintentos default: **2 reintentos = 3 intentos totales**
- [ ] Tiempo entre reintentos: ~1 min, luego ~2 min (no configurable)
- [ ] Max event age default: **6 horas** (configurable 60s - 6h)
- [ ] Después de fallar todo → DLQ o Destinations (si no → descartado)
- [ ] **DLQ**: SQS o SNS, solo fallos
- [ ] **Destinations** (recomendado): SQS, SNS, Lambda, EventBridge — OnSuccess + OnFailure
- [ ] Servicios push asíncronos clave: **S3, SNS, EventBridge**
- [ ] Diferencia vs Event Source Mapping: asíncrono = servicio PUSH; ESM = Lambda POLL
