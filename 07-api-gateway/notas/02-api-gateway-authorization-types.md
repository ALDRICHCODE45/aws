# API Gateway — Tipos de Autorización

| Tipo | Cuándo usar |
| ---- | ----------- |
| **NONE** | Endpoint público |
| **AWS_IAM** | Cross-account / servicios AWS / usuarios IAM con credenciales |
| **Cognito User Pool** | Usuarios finales de app (móvil/web) |
| **Lambda Authorizer** | Token custom, JWT propio, OAuth con IdP externo |

## Acciones IAM para API Gateway

| Acción | Para qué |
| ------ | -------- |
| `execute-api:Invoke` | Invocar endpoint |
| `execute-api:InvalidateCache` | Invalidar cache vía header |
| `execute-api:ManageConnections` | WebSocket |

> ⚠️ `apigateway:Invoke` y `api-gateway:Invoke` **NO existen**. La correcta es `execute-api:Invoke`.

## Disparadores → tipo

- "Cross-account" + "rol IAM" + "sin complejidad" → **AWS_IAM** + resource policy
- "Usuarios de app móvil/web" → **Cognito User Pool**
- "JWT propio" / "Auth0 / Okta" / "token custom" → **Lambda Authorizer**
- "Endpoint público" → **NONE**

## Trampa recurrente

**API Keys NO autentican, solo identifican.** Si la pregunta dice "autenticar otra cuenta/usuario" y la opción es API Keys → descarte.

API Keys + Usage Plans = tiers comerciales (Free/Pro), NO autenticación.

## Códigos HTTP en API Gateway — mapeo

| Código | Causa |
| ------ | ----- |
| **429** Too Many Requests | **Throttling** (rate limit / cuota usage plan) |
| **502** Bad Gateway | Lambda devolvió respuesta mal formada |
| **503** Service Unavailable | Servicio AWS caído (raro) |
| **504** Gateway Timeout | **Backend > 29s** (timeout de API GW) |
| **403** Forbidden | API Key inválida / sin permiso al stage/método |
| **401** Unauthorized | Token mal o no provisto |

## Timeout límite

API Gateway tiene **timeout fijo de 29 segundos** para llamar al backend.

| | Timeout máximo |
| -- | -------------- |
| Lambda función sola | 15 minutos |
| **Lambda detrás de API GW** | **29 segundos** |

Síntomas típicos en picos de tráfico:
- Cold starts → Lambdas más lentas → 504.
- DynamoDB throttling → retries → Lambda > 29s → 504.

## Trampa común

"Aumento de tráfico + 504" → suena a throttling pero **throttling = 429**. 504 es SIEMPRE timeout.
