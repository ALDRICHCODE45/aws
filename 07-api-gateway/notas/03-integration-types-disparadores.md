# API Gateway — Integration types (modo examen)

## Tabla rápida

| Tipo | Backend | Transformación | Cuándo |
| ---- | ------- | -------------- | ------ |
| **AWS** custom | Servicio AWS | ✅ Sí | Lambda/SQS/DDB con transformación |
| **AWS_PROXY** | **Solo Lambda** | ❌ NO | Lambda recibe evento crudo |
| **HTTP** custom | HTTP externo | ✅ Sí | Microservicios HTTP + transformación |
| **HTTP_PROXY** | HTTP externo | ❌ NO | Proxy puro a HTTP |

## Disparadores

| Pregunta dice | Integration |
| ------------- | ----------- |
| "transformar" / "mapping templates" / "control sobre headers/body/params" | **AWS** o **HTTP** (custom) |
| "microservicios" + transformación | **HTTP** custom |
| "Lambda" + transformación | **AWS** custom |
| "evento crudo" / "sin transformación" / "proxy" | **`_PROXY`** |

## Regla mental

- `_PROXY` → pasa tal cual, sin tocar. Si la pregunta pide transformar → descarte.
- `AWS_PROXY` es **exclusivo de Lambda**. No sirve para SQS/DDB/S3.
- Microservicios HTTP + transformación → **HTTP** (no AWS).
