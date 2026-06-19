# SNS vs SQS + Lambda Destinations

## SNS vs SQS

| | SNS | SQS |
| -- | --- | --- |
| Patrón | **Pub/Sub** (fan-out) | Cola 1-a-1 |
| Suscriptores | **Múltiples** reciben lo mismo | Solo UNO procesa cada mensaje |
| Push/Pull | Push | Pull |

## Disparadores

| Pregunta dice | Servicio |
| ------------- | -------- |
| "pub/sub" / "fan-out" / "múltiples suscriptores reciben lo mismo" | **SNS** |
| "notificar a varios servicios simultáneamente" | **SNS** |
| "cola de tareas / un consumidor procesa cada uno" | **SQS** |
| "fan-out + persistencia por suscriptor" | **SNS + SQS** combinados |

## Lambda Destinations

Rutea resultado de Lambda **asíncrona** sin código adicional.

| Tipo | Destinos válidos |
| ---- | ---------------- |
| `on-success` | SQS, SNS, Lambda, EventBridge |
| `on-failure` | SQS, SNS, Lambda, EventBridge |

> ⚠️ Solo invocaciones **asíncronas** (S3, SNS triggers, EventBridge). NO síncronas (API GW).

## Trampa

"Notificar a múltiples servicios downstream" + "sin código adicional" → **Lambda Destination on-success + SNS** (porque SNS hace fan-out). Si la opción usa SQS para esto → mal: SQS no es pub/sub.
