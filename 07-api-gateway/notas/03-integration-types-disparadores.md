# API Gateway — Integration types (modo examen)

## Tabla rápida

| Tipo            | Backend         | Transformación | Cuándo                               |
| --------------- | --------------- | -------------- | ------------------------------------ |
| **AWS** custom  | Servicio AWS    | ✅ Sí          | Lambda/SQS/DDB con transformación    |
| **AWS_PROXY**   | **Solo Lambda** | ❌ NO          | Lambda recibe evento crudo           |
| **HTTP** custom | HTTP externo    | ✅ Sí          | Microservicios HTTP + transformación |
| **HTTP_PROXY**  | HTTP externo    | ❌ NO          | Proxy puro a HTTP                    |

## Disparadores

| Pregunta dice                                                             | Integration                 |
| ------------------------------------------------------------------------- | --------------------------- |
| "transformar" / "mapping templates" / "control sobre headers/body/params" | **AWS** o **HTTP** (custom) |
| "microservicios" + transformación                                         | **HTTP** custom             |
| "Lambda" + transformación                                                 | **AWS** custom              |
| "evento crudo" / "sin transformación" / "proxy"                           | **`_PROXY`**                |

## Regla mental

- `_PROXY` → pasa tal cual, sin tocar. Si la pregunta pide transformar → descarte.
- `AWS_PROXY` es **exclusivo de Lambda**. No sirve para SQS/DDB/S3.
- Microservicios HTTP + transformación → **HTTP** (no AWS).

## Decisión en 2 ejes (cruzá SIEMPRE)

1. ¿Backend HTTP o servicio AWS?
2. ¿Pass-through (sin manipulación) o custom (con mapping)?

|               | Pass-through                | Custom (mapping) |
| ------------- | --------------------------- | ---------------- |
| HTTP endpoint | **HTTP_PROXY**              | **HTTP**         |
| Servicio AWS  | **AWS_PROXY** (solo Lambda) | **AWS**          |

## Caso que fallé (Beanstalk + API Gateway)

- Backend = Beanstalk multicontenedor = **endpoint HTTP** (NO es Lambda → descartar AWS/AWS_PROXY).
- "redirigir DIRECTAMENTE al backend SIN manipulación" → pass-through → **HTTP_PROXY**.
- Elegí `HTTP` (custom) = lo CONTRARIO (custom = con mapping). Mal.
- Ruido a ignorar: "salvo casos puntuales como caracteres no soportados" NO cambia
  la respuesta; HTTP_PROXY igual lo maneja. No te desvíes por la excepción menor.

## Pregunta de prueba

Querés exponer por API Gateway una app en Beanstalk (endpoint HTTP), redirigiendo
las solicitudes DIRECTAMENTE al backend sin manipulación. ¿Qué tipo de integración?

A) `AWS_PROXY`
B) `HTTP_PROXY`
C) `HTTP`
D) `AWS`

<details><summary>Respuesta</summary>

**B** (`HTTP_PROXY`): backend HTTP + pass-through (sin mapping).
Cuándo sería cada una:
- **HTTP** → backend HTTP pero CON transformación (mapping templates).
- **AWS_PROXY** → solo Lambda (pass-through).
- **AWS** → invocar un servicio AWS (SQS/DynamoDB/etc.) con mapping.
</details>
