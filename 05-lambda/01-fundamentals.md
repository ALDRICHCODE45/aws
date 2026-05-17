# Lambda — Fundamentos

Qué es, modelo de recursos, pricing. Base para todo lo demás.

---

## ¿Qué es Lambda?

Compute **serverless**: ejecutás código sin gestionar servidores. AWS aprovisiona, parchea, escala y monitorea.

### Pilares
- **Serverless** → vos solo subís código
- **Event-driven** → se ejecuta en respuesta a un trigger
- **Pay-per-use** → request + duración (redondeo a 1 ms)
- **Integración nativa** → S3, DynamoDB, API Gateway, SNS, SQS, EventBridge, etc.
- **Escalado automático** → de 0 a miles concurrentes sin configurar

---

## Modelo de recursos (CRÍTICO)

| Recurso | Rango | Detalle |
|---|---|---|
| **Memoria** | 128 MB → 10,240 MB | Incrementos de 1 MB |
| **CPU** | No se configura | Escala **proporcional a memoria** |
| **Red** | No se configura | Escala **proporcional a memoria** |
| **Timeout** | 1 s → **900 s (15 min)** | Límite duro |
| **Ephemeral storage (/tmp)** | 512 MB → 10,240 MB | 512 MB gratis |
| **Arquitecturas** | x86_64 / **arm64 (Graviton2)** | ARM ~20% más barato |

> 🎯 **Truco examen**: "necesito más CPU" → **aumentar memoria**. No hay slider de CPU.

---

## Pricing

### Free tier (permanente)
- **1M requests/mes**
- **400,000 GB-segundos/mes**

> ⚠️ Unidad: **GB-segundos** = memoria (GB) × duración (s). NO "GB" a secas.

| Memoria | Segundos cubiertos por free tier |
|---|---|
| 512 MB | 800,000 s |
| 1 GB | 400,000 s |
| 2 GB | 200,000 s |
| 10 GB | 40,000 s |

### Post free tier
- **Requests**: $0.20 por millón
- **Duración**: depende de memoria + arquitectura

---

## Casos de uso típicos
- Procesamiento de archivos (S3 trigger)
- Backends de APIs (API Gateway → Lambda)
- Stream processing (Kinesis, DynamoDB Streams)
- Cron jobs (EventBridge)
- ETL event-driven
- Mobile/IoT backends

---

## Checklist examen

- [ ] Serverless = vos NO gestionás servidores
- [ ] Memoria: **128 MB a 10,240 MB**
- [ ] Timeout máximo: **15 min (900 s)**
- [ ] CPU/red escalan **proporcional a memoria**
- [ ] Free tier: 1M requests + **400K GB-s** (permanente)
- [ ] x86 y **ARM (Graviton2 — más barato)**
- [ ] Pago por request + duración (redondeo 1 ms)
