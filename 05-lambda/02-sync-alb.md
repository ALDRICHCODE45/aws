# Lambda — Invocación síncrona vía ALB

Cómo exponer Lambda como endpoint HTTP/HTTPS detrás de un Application Load Balancer.

---

## Concepto

```
Cliente HTTP/S
   ↓
ALB (listener :80 o :443)
   ↓
Listener Rule (matchea path/header/host)
   ↓
Target Group (tipo: lambda)
   ↓
Lambda Function (1 sola por target group)
```

Lambda + Load Balancer = **solo ALB**. NLB, GLB, CLB ❌ NO soportan Lambda.

---

## Target Group

Es la **lista de cosas que reciben tráfico** del ALB. El ALB nunca habla directo con recursos; siempre pasa por un target group.

| Tipo | Contiene |
|---|---|
| `instance` | EC2 |
| `ip` | IPs (incluso on-premises) |
| **`lambda`** | **1 sola función** |
| `alb` | Otro ALB |

> ⚠️ Target group tipo lambda = **1 función**. Para varias Lambdas: un target group por cada una + listener rules.

---

## Comunicación ALB ↔ Lambda

| Paso | Qué pasa |
|---|---|
| Cliente → ALB | HTTP/S normal |
| ALB → Lambda | ALB convierte request a **JSON** (`event`) |
| Lambda → ALB | Devuelve **JSON** (`statusCode`, `headers`, `body`) |
| ALB → Cliente | ALB convierte JSON a respuesta HTTP |

Invocación **SÍNCRONA**: el ALB espera la respuesta.

### Event (request)
```json
{
  "httpMethod": "GET",
  "path": "/users",
  "queryStringParameters": { "id": "42" },
  "headers": { "host": "...", "user-agent": "..." },
  "body": "",
  "isBase64Encoded": false
}
```

### Response esperada
```json
{
  "statusCode": 200,
  "headers": { "Content-Type": "application/json" },
  "body": "{\"hello\":\"world\"}",
  "isBase64Encoded": false
}
```

Headers que ALB inyecta siempre: `X-Amzn-Trace-Id`, `X-Forwarded-For`, `X-Forwarded-Port`, `X-Forwarded-Proto`.

---

## Multi-value headers

Por defecto **OFF**. Si un header/query se repite, ALB te pasa solo el **último valor**.

| Modo | Campo en event |
|---|---|
| OFF (default) | `headers`, `queryStringParameters` (string) |
| ON | `multiValueHeaders`, `multiValueQueryStringParameters` (array) |

Se activa en **attributes del target group**.

---

## Límites

| Cosa | Valor |
|---|---|
| Request body máx | **1 MB** |
| Response máx | **1 MB** |
| WebSockets | ❌ NO (HTTP 400) |
| Funciones por target group | 1 |
| Lambda + target group | Misma cuenta + región |

Health checks: **OFF por defecto**. Cuando se activan, cuentan como invocación facturada.

---

## ALB vs API Gateway

| Necesidad | Elegir |
|---|---|
| API REST completa, throttling, API keys | **API Gateway** |
| Auth con Cognito | **API Gateway** |
| **WebSockets** | **API Gateway** |
| Caching, transformations | **API Gateway** |
| HTTP simple detrás de ALB existente | **ALB** |
| Routing path/header básico | **ALB** |

---

## Checklist examen

- [ ] Solo **ALB** soporta Lambda (no NLB, no CLB)
- [ ] Target group tipo lambda = **1 función**
- [ ] Invocación **síncrona**, intercambia **JSON**
- [ ] Multi-value headers **OFF por defecto**
- [ ] Health checks **OFF por defecto**
- [ ] **No WebSockets** → usar API Gateway
- [ ] Request/Response máx **1 MB**
- [ ] Misma cuenta + misma región
