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
