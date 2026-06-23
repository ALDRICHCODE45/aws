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

## PISTA REGALADA: el JSON del evento

Si el enunciado muestra esta estructura de entrada → es **Lambda proxy (`AWS_PROXY`)**, sí o sí:

```
resource, path, httpMethod, headers, multiValueHeaders,
queryStringParameters, multiValueQueryStringParameters,
pathParameters, stageVariables, requestContext, body, isBase64Encoded
```

Ese objeto ES el evento que `AWS_PROXY` le pasa a Lambda con todo el request crudo.
Reconocerlo = respuesta instantánea.

## Caso que fallé #2 (Lambda + API Gateway) — eje 1 mal

- Backend = **función Lambda** → familia `AWS_*`, NUNCA `HTTP_*`.
- "rápida, mínima config", recibe el request completo crudo → `_PROXY`.
- Cruce → **`AWS_PROXY`** (Lambda proxy). Elegí `HTTP_PROXY` = acerté el eje 2 (proxy)
  pero erré el eje 1 (backend). LECCIÓN: agarrar "proxy" y frenar NO alcanza. CRUZÁ LOS DOS EJES.

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

## Pregunta de prueba #2 (caso Lambda — eje 1)

Una **función Lambda** debe integrarse con API Gateway y recibir el request completo
(headers, query params, body, requestContext) en un JSON, de forma **rápida y con mínima
configuración**. ¿Qué tipo de integración?

A) Integración proxy con HTTP (`HTTP_PROXY`)
B) Integración proxy con Lambda (`AWS_PROXY`)
C) Integración personalizada con Lambda (Lambda custom)
D) Integración personalizada con HTTP (HTTP custom)

<details><summary>Respuesta</summary>

**B** (`AWS_PROXY` = Lambda proxy). Cruzá: backend Lambda → familia AWS; mínima config + request crudo → `_PROXY`.
- **A** `HTTP_PROXY` → backend es endpoint HTTP, NO Lambda (eje 1 mal: la trampa donde caí).
- **C / D** custom → usan mapping templates = MÁS config (descartados por "mínima configuración").
</details>
