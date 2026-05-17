# Lambda — EventBridge (ex CloudWatch Events)

Servicio bus de eventos serverless. Antes se llamaba **CloudWatch Events** (mismo servicio, nombre viejo).

> ⚠️ Examen: "CloudWatch Events" y "EventBridge" = **mismo servicio**. Son intercambiables.

---

## Dos patrones para invocar Lambda

### Patrón A — Schedule (cron job)

```
EventBridge Rule (cron / rate)
   ↓ AWS dispara automáticamente
Lambda Function
```

Ejemplos: limpieza nocturna, reportes diarios, health checks periódicos.

### Patrón B — Event pattern (reaccionar a cambios)

```
Servicio AWS (ej: CodePipeline cambia estado)
   ↓ emite evento al bus
EventBridge Rule (event pattern matchea)
   ↓
Lambda Function
```

Ejemplos: pipeline falló → notificar Slack; EC2 creada → registrar; login root → alerta.

---

## Sintaxis schedule

| Tipo | Ejemplo | Significado |
|---|---|---|
| `rate()` | `rate(1 hour)` | Cada hora |
| `rate()` | `rate(5 minutes)` | Cada 5 min |
| `cron()` | `cron(0 12 * * ? *)` | Diario 12:00 UTC |
| `cron()` | `cron(0 9 ? * MON-FRI *)` | Lun-Vie 9am UTC |

⚠️ Cron de AWS = **6 campos** (no 5 como Unix).
⚠️ Siempre **UTC**.

---

## EventBridge → otros targets (no solo Lambda)

EventBridge puede enviar eventos a 20+ destinos:
- **Lambda**, **SQS**, **SNS**
- **Step Functions**, **ECS Tasks**
- **Kinesis** (Streams / Firehose)
- **API Destinations** (HTTP externo)
- Otro event bus (cross-account / cross-region)

> 🎯 Examen: "correr Step Function cada día" → **EventBridge Schedule + Step Function**. No necesitás Lambda intermedia.

---

## Tipos de Event Bus

| Tipo | Para qué |
|---|---|
| **Default** | Eventos de servicios AWS de la cuenta (automático) |
| **Custom** | Eventos de tus apps |
| **Partner** | Eventos de SaaS (Datadog, Zendesk, etc.) |

---

## EventBridge → Lambda = ASÍNCRONA

Aplica todo lo de invocación asíncrona:
- Reintentos automáticos
- DLQ / Destinations
- **Idempotencia obligatoria**

---

## EventBridge Scheduler (servicio nuevo, 2022)

Sub-servicio aparte, más potente que las rules cron tradicionales:
- One-time schedules + recurring
- Hasta **185 reintentos**, 24 h event age
- Time windows flexibles

Recomendado para casos nuevos.

---

## Checklist examen

- [ ] EventBridge = CloudWatch Events (mismo servicio)
- [ ] Dos patrones: **schedule** (cron) y **event pattern** (reacción)
- [ ] Cron AWS = **6 campos**, en **UTC**
- [ ] `rate(N units)` o `cron(...)`
- [ ] 20+ targets (Lambda, SQS, SNS, Step Functions, ECS...)
- [ ] Para cron + Step Functions/ECS → EventBridge directo, no necesitás Lambda
- [ ] Default bus = eventos AWS; Custom = apps propias; Partner = SaaS
- [ ] EventBridge → Lambda es **asíncrono** (aplica idempotencia)
