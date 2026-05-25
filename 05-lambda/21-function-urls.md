# Lambda Function URLs

## ¿Qué son?

**Endpoint HTTPS dedicado** para invocar tu función Lambda directamente desde internet, sin API Gateway.

### Características:
- **URL única y permanente** (nunca cambia)
- Formato: `https://<url-id>.lambda-url.<region>.on.aws`
- Soporta **dual-stack IPv4 & IPv6**
- **Solo acceso público** (NO soporta PrivateLink)
- Se puede aplicar a **alias** o **$LATEST** (NO a versiones numeradas)
- Configurable vía **consola AWS** o **AWS API**
- Acelera función usando **concurrencia reservada**

---

## Tipos de Autenticación

### AuthType: NONE
```json
{
  "lambda:FunctionUrlAuthType": "NONE"
}
```
- **Acceso público** y no autenticado
- Cualquiera puede invocar la función desde internet
- La **política basada en recursos** debe estar en **Effect: "Allow"**

### AuthType: AWS_IAM
```json
{
  "lambda:FunctionUrlAuthType": "AWS_IAM"
}
```
- Requiere **credenciales AWS** para invocar
- Se evalúan **política de identidad** Y **política basada en recursos**

---

## Seguridad con AWS_IAM

### Evaluación de políticas:

**Misma cuenta AWS:**
- **Política de identidad** O **Política basada en recursos** = ALLOW
- Una de las dos es suficiente

**Cuenta cruzada:**
- **Política de identidad** Y **Política basada en recursos** = ALLOW  
- Ambas son requeridas

### Política basada en recursos (en Lambda):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "lambda:InvokeFunctionUrl",
      "Resource": "arn:aws:lambda:region:account:function:my-function",
      "Condition": {
        "StringEquals": {
          "lambda:FunctionUrlAuthType": "AWS_IAM"
        }
      }
    }
  ]
}
```

### Política basada en identidad (en IAM principal):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "lambda:InvokeFunctionUrl",
      "Resource": "arn:aws:lambda:region:account:function:my-function"
    }
  ]
}
```

---

## CORS (Cross-Origin Resource Sharing)

Configuración para **llamadas desde browser**:
```json
{
  "AllowOrigins": ["https://example.com"],
  "AllowMethods": ["GET", "POST", "PUT"],
  "AllowHeaders": ["content-type", "x-amz-date"],
  "ExposeHeaders": ["x-amz-request-id"],
  "MaxAge": 86400,
  "AllowCredentials": true
}
```

### Diagrama de uso con CORS:
```
Browser (example.com) → CloudFront → Lambda Function URL
                    ↓
             (CORS validation)
```

---

## Casos de uso

### ✅ Cuándo usar Function URLs:
- **Webhook endpoints** simples
- **APIs ligeras** sin necesidad de API Gateway features
- **Funciones públicas** (health checks, webhooks de terceros)
- **Microservicios** independientes con una sola función

### ❌ Cuándo NO usar:
- Necesitás **rate limiting, throttling**
- Requiere **custom domains**
- Necesitás **caching, transformaciones**
- **APIs complejas** con múltiples endpoints
→ **Usar API Gateway** en estos casos

---

## Para el examen

**Pregunta típica:** "Invocar Lambda desde internet sin API Gateway"
**Respuesta:** Lambda Function URLs

**Key facts:**
- **AuthType NONE** = acceso público (política basada en recursos en Allow)
- **AuthType AWS_IAM** = credenciales AWS requeridas
- **Misma cuenta**: política identidad O recursos = ALLOW
- **Cuenta cruzada**: política identidad Y recursos = ALLOW
- **No soporta PrivateLink** (solo acceso público)
- **CORS** configurable para llamadas desde browser
- **Alternativa a API Gateway** para casos simples