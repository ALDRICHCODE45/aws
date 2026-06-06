# API Gateway — Tipos de Integración

## La idea base

API Gateway es un **traductor** entre el cliente (que habla HTTP/JSON) y el backend (que habla en su propio formato). El **tipo de integración** define **CÓMO** API Gateway habla con ese backend.

> Analogía: si API Gateway es un recepcionista políglota, el "tipo de integración" es el idioma que usa para hablar con cada habitación del edificio.

## Los 5 tipos de integración

| Tipo                         | Backend                                | Necesita mapping template | Caso de uso                                  |
| ---------------------------- | -------------------------------------- | ------------------------- | -------------------------------------------- |
| **MOCK**                     | Ninguno                                | No                        | Respuesta fake (testing, health)             |
| **HTTP**                     | Servidor HTTP cualquiera               | Sí                        | Backend HTTP existente que querés "envolver" |
| **HTTP_PROXY**               | Servidor HTTP cualquiera               | No                        | Pasar request crudo al backend HTTP          |
| **AWS**                      | Servicio AWS (SQS, DynamoDB, S3, etc.) **o Lambda custom** | Sí        | Exponer un servicio AWS como REST sin Lambda, o Lambda con mapping |
| **AWS_PROXY** (Lambda Proxy) | **Solo Lambda** (no otros servicios AWS) | No                      | El 95% de los casos con Lambda               |

## 1. MOCK — Respuesta fake

API Gateway **NO llama a ningún backend**. Devuelve una respuesta que vos hardcodeás.

**Casos de uso:**

- Health checks (`/health` que devuelve siempre `{"status":"ok"}`).
- Frontend que necesita el endpoint para desarrollar mientras el backend no existe.
- Simular escenarios de error para testing.
- **CORS desde la consola** (ver abajo).

> Analogía: el mozo te trae el plato sin pasar por la cocina.

> **TRAMPA DE EXAMEN — MOCK + CORS**: cuando habilitás CORS desde la consola de API Gateway, el método `OPTIONS` se crea automáticamente como una integración **MOCK** que devuelve los headers CORS (`Access-Control-Allow-Origin`, `-Methods`, `-Headers`). No hay backend que llamar — los headers son fake response.
>
> Cita textual de la doc: _"the API Gateway console integrates the `OPTIONS` method to support CORS with a mock integration"_.
>
> Si te preguntan "¿qué tipo de integración usa el método OPTIONS generado por la consola al habilitar CORS?" → **MOCK**.

## 2. HTTP — HTTP backend con transformación

API Gateway llama a un **endpoint HTTP cualquiera** (puede ser un ALB, un EC2, un servidor on-premise vía Direct Connect, lo que sea).

**Pero vos controlás la traducción**: configurás **mapping templates** para transformar:

- El request del cliente → al formato que espera el backend HTTP.
- La respuesta del backend → al formato que querés devolver al cliente.

**Útil cuando:**

- El backend espera un payload distinto del que manda el cliente.
- Querés filtrar/ocultar campos del backend antes de devolverlos.
- Querés agregar headers, cambiar paths, etc.

## 3. HTTP_PROXY — HTTP backend "passthrough"

Igual que HTTP, pero **sin transformación**. API Gateway **reenvía el request tal cual** al backend HTTP y devuelve la respuesta tal cual al cliente.

**Útil cuando:**

- El backend ya entiende el formato del cliente.
- No querés mantener mapping templates.
- Querés que API Gateway sea solo una "fachada" (auth, throttling, cache) sin meterse con el payload.

> Regla mental: **PROXY = passthrough (sin mapping)**. **Sin PROXY = con mapping template**.

## 4. AWS — Invocación directa a servicio AWS (con mapping)

**ESTE ES EL TIPO MÁS IMPORTANTE para diferenciarse en el examen.**

API Gateway llama **DIRECTAMENTE a un servicio AWS** (SQS, DynamoDB, S3, Step Functions, Kinesis, SNS, etc.) **SIN Lambda en el medio**.

> **Sutileza de la doc oficial**: el tipo `AWS` también cubre la llamada **Lambda Custom Integration**. Es Lambda + mapping templates (vos transformás el request antes de que Lambda lo vea, y la response antes de devolverla al cliente). AWS lo trata como un caso especial de la integración `AWS`, porque el "servicio AWS" que se invoca es la acción `Invoke` de Lambda.
>
> O sea: cuando ves `type: AWS` puede ser (a) cualquier servicio AWS no-Lambda con mapping, o (b) Lambda con mapping (custom). Las dos comparten el mismo `type` value.

**Cómo funciona:**

- El cliente manda un request REST normal (ej: `POST /orders {"item":"pizza"}`).
- API Gateway usa un **mapping template (VTL)** para traducir ese JSON al formato que espera el servicio AWS.
- El servicio AWS responde, y otro mapping template traduce esa respuesta al formato que devolvés al cliente.

**Ejemplo SQS (el del dibujo):**

```
Cliente → POST /orders  {"item":"pizza"}
          ↓
API Gateway (mapping template VTL)
          ↓ traduce a:
SQS  ← Action=SendMessage&MessageBody={"item":"pizza"}&QueueUrl=...
```

**Beneficios:**

- **Ahorra costo de Lambda** (no hay invocación, no hay duración).
- **Menos latencia** (un hop menos).
- **Menos código que mantener** (no hay handler que actualizar).

**Limitaciones:**

- Necesitás conocer **VTL** (Velocity Template Language) para los mapping templates.
- Curva de aprendizaje más alta.
- Difícil de debuggear si el mapping falla.

> **Pregunta típica de examen**: _"Querés exponer un endpoint REST que mande mensajes a SQS sin pagar por Lambda. ¿Cómo lo hacés?"_
>
> Respuesta: **API Gateway con integración tipo AWS directa a SQS, usando mapping templates VTL.**

## 5. AWS_PROXY (Lambda Proxy) — la más común con Lambda

API Gateway pasa **TODO el request crudo** a Lambda en un objeto estándar (`event`), y Lambda devuelve un objeto con un formato estándar que API Gateway traduce a HTTP response.

**Sin mapping templates**. Lambda decide TODO en su código.

> **MUY IMPORTANTE — trampa de examen**: `AWS_PROXY` es **EXCLUSIVO de Lambda**. No podés usarlo para SQS, DynamoDB, S3, ni ningún otro servicio AWS. Para esos servicios, obligatoriamente `AWS` (custom) con mapping templates.
>
> Cita textual de la doc: _"This is the preferred integration type to call a Lambda function through API Gateway and is **not applicable to any other AWS service actions**, including Lambda actions other than the function-invoking action."_

**Estructura del event que recibe Lambda:**

```text
{
  "httpMethod": "POST",
  "path": "/orders",
  "headers": { ... },
  "queryStringParameters": { ... },
  "body": "...",
  "stageVariables": { ... },
  "requestContext": { ... }
}
```

**Lambda debe devolver:**

```text
{
  "statusCode": 200,
  "headers": { ... },
  "body": "..."
}
```

**Por qué es la más común:**

- Cero configuración en API Gateway.
- Toda la lógica está en código (más fácil de versionar, testear, debuggear).
- Lambda tiene control total del request/response.

> Regla mental: si trabajás con Lambda + API Gateway, **el 95% de las veces es AWS_PROXY**.

## HTTP vs AWS — la confusión clásica

A primera vista parecen idénticos: los dos usan mapping templates, los dos transforman, los dos pueden llamar a "algo externo". **Pero NO son lo mismo**, y la diferencia es CRÍTICA para examen.

### La diferencia clave: a quién llaman y cómo

|                         | **HTTP**                                         | **AWS**                                                                         |
| ----------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------- |
| **Backend**             | Cualquier servidor HTTP                          | Un **servicio AWS** específico                                                  |
| **Cómo lo invoca**      | URL HTTP normal (`https://miapi.com/users`)      | **AWS API firmada con SigV4**                                                   |
| **Autenticación**       | La que tenga el servidor (API key, OAuth, nada)  | **IAM Role** de API Gateway con permisos al servicio                            |
| **Red**                 | Backend debe ser accesible (internet o VPC Link) | Red interna de AWS, no sale a internet                                          |
| **Formato del payload** | Vos decidís cómo lo arma el mapping              | **Formato EXACTO** que el servicio AWS espera (definido por AWS, no negociable) |
| **Ejemplos**            | EC2, ALB, on-premise, Stripe, GitHub API         | SQS, DynamoDB, S3, Kinesis, Step Functions, SNS                                 |

### Analogía para que quede grabado

> **HTTP** es como llamar a un **número de teléfono cualquiera**. Marcás, hablás, la otra persona decide cómo te atiende. El "protocolo" es libre.
>
> **AWS** es como entrar a una **oficina interna de AWS con credencial IAM**. No podés entrar sin la credencial firmada con SigV4, y adentro hablás un protocolo específico que AWS define.

### Lo que parece igual y NO lo es

Sí, los dos:

- Necesitan mapping templates VTL.
- Transforman request/response.
- No usan Lambda.

**Pero el "cómo invocan al backend" es radicalmente distinto**, y eso define 3 cosas críticas:

1. **Autenticación**:
   - **HTTP** → vos configurás cómo se autentica (API key, OAuth, lo que sea el backend pida).
   - **AWS** → API Gateway necesita un **IAM Role** con permisos al servicio (ej: `sqs:SendMessage`, `dynamodb:PutItem`).

   > Pregunta típica de examen: _"API Gateway no puede mandar mensajes a SQS, ¿qué falta?"_ → **IAM Role con permisos `sqs:SendMessage`**. Solo aplica si es integración **AWS**, no HTTP.

2. **Conectividad**:
   - **HTTP** → el backend tiene que ser accesible desde internet (o vía VPC Link para llegar a VPC privada).
   - **AWS** → API Gateway llega al servicio por la red interna de AWS, sin tocar internet.

3. **Formato del payload**:
   - **HTTP** → vos definís el formato que tu backend espera (vos sos el dueño).
   - **AWS** → tenés que cumplir el formato exacto que el servicio AWS espera. Ejemplo SQS: `Action=SendMessage&MessageBody=...`. Si te equivocás un campo, AWS rechaza el request.

### Casos de uso clásicos

| Necesidad                                                             | Tipo correcto |
| --------------------------------------------------------------------- | ------------- |
| Exponer una API legacy que corre en EC2 con transformación de payload | **HTTP**      |
| Envolver Stripe / una API de terceros con auth custom                 | **HTTP**      |
| Wrapping de un backend on-premise (vía Direct Connect)                | **HTTP**      |
| Mandar mensajes directo a SQS sin Lambda                              | **AWS**       |
| Leer/escribir directo en DynamoDB sin Lambda                          | **AWS**       |
| Subir archivos directo a S3 sin Lambda                                | **AWS**       |
| Disparar una Step Function desde una API REST                         | **AWS**       |

### Resumen mental

> **HTTP** = backend "externo" (servidor HTTP cualquiera, propio o de terceros, vos controlás el protocolo).
> **AWS** = backend "interno de AWS" (servicios AWS con IAM Role y SigV4, el protocolo lo define AWS).

Los dos transforman con mapping templates, pero la **naturaleza del backend** es completamente distinta.

## PROXY vs NO-PROXY — la regla mental

|                         | NO-PROXY (HTTP, AWS)                   | PROXY (HTTP_PROXY, AWS_PROXY)  |
| ----------------------- | -------------------------------------- | ------------------------------ |
| **Mapping template**    | Sí, obligatorio                        | No, passthrough                |
| **Control del payload** | API Gateway transforma                 | Backend recibe todo crudo      |
| **Mantenimiento**       | Mapping templates en VTL               | Código del backend             |
| **Cuándo usarlo**       | Transformar/filtrar request o response | Backend ya entiende el formato |

## Tabla comparativa rápida

| Tipo       | Backend                        | Quién traduce                | Cuándo usarlo                             |
| ---------- | ------------------------------ | ---------------------------- | ----------------------------------------- |
| MOCK       | —                              | API Gateway (respuesta fake) | Testing, health checks                    |
| HTTP       | HTTP cualquiera                | Mapping template VTL         | Backend HTTP que necesita transformación  |
| HTTP_PROXY | HTTP cualquiera                | Nadie (passthrough)          | Backend HTTP que ya entiende el formato   |
| AWS        | Servicio AWS (SQS, DDB, S3...) | Mapping template VTL         | Exponer servicio AWS como REST sin Lambda |
| AWS_PROXY  | Lambda                         | Lambda en código             | Caso clásico Lambda + API Gateway         |

## Trampas de examen

1. **PROXY = sin mapping template, passthrough**. SIN PROXY = mapping template VTL obligatorio.
2. **AWS (sin proxy)** sirve para invocar **servicios AWS directamente sin Lambda** (SQS, DynamoDB, S3, etc.). **Ahorra costo y latencia**.
3. **AWS_PROXY = Lambda Proxy Integration**. Es el default para Lambda.
4. **AWS_PROXY es SOLO para Lambda**. Para SQS/DynamoDB/S3/etc. usás `AWS` (custom), nunca `AWS_PROXY`.
5. **Lambda Custom Integration = `type: AWS`** (no `AWS_PROXY`). Es Lambda + mapping templates, útil cuando querés transformar antes de que Lambda vea el request.
6. **HTTP vs HTTP_PROXY**: solo cambia si hay transformación con mapping template o no.
7. **MOCK** = respuesta fake sin backend. Usos: testing, health checks, **y el método OPTIONS de CORS** cuando lo habilitás desde la consola.
8. Mapping templates usan **VTL (Velocity Template Language)**. No es JSON, es un lenguaje aparte.

## Auto-test

1. Querés exponer un endpoint REST que mande mensajes directo a SQS sin pasar por Lambda. ¿Qué tipo de integración usás?
2. Tu Lambda recibe el request completo (headers, body, path) en un objeto `event` estándar. ¿Qué tipo de integración tenés?
3. ¿Cuál es la diferencia entre HTTP y HTTP_PROXY?
4. Querés que `/health` devuelva siempre `{"status":"ok"}` sin llamar a ningún backend. ¿Qué tipo de integración?
5. En qué tipos de integración necesitás escribir mapping templates en VTL?
6. ¿Cuál es el tipo de integración más común cuando trabajás con Lambda?
7. ¿Podés usar `AWS_PROXY` para integrar API Gateway con DynamoDB directamente?
8. Tenés Lambda como backend pero querés transformar el request con un mapping template antes de que Lambda lo vea. ¿Qué tipo de integración configurás?
9. Habilitás CORS desde la consola de API Gateway. ¿Qué tipo de integración usa el método `OPTIONS` que se crea automáticamente?

<details>
<summary>Respuestas</summary>

1. **AWS** (sin proxy). Con mapping template VTL que traduce el JSON del cliente al formato de SQS. Ahorra costo de Lambda y baja latencia.
2. **AWS_PROXY** (Lambda Proxy Integration). API Gateway pasa todo crudo en `event`, y Lambda devuelve `{statusCode, headers, body}`.
3. **HTTP** usa mapping template para transformar request/response. **HTTP_PROXY** es passthrough: pasa el request crudo al backend y devuelve la respuesta cruda al cliente.
4. **MOCK**. No llama a ningún backend, devuelve la respuesta hardcodeada.
5. En los tipos **SIN PROXY**: **HTTP** y **AWS**.
6. **AWS_PROXY** (Lambda Proxy). Es el default y el más usado.
7. **NO**. `AWS_PROXY` es exclusivo de Lambda. Para DynamoDB (o cualquier otro servicio AWS) usás `AWS` (custom) con mapping templates VTL.
8. **AWS** (Lambda Custom Integration). El tipo es `AWS`, no `AWS_PROXY`. Te da control total del request/response con mapping templates.
9. **MOCK**. La consola crea el `OPTIONS` como integración MOCK que devuelve los headers CORS hardcodeados, sin llamar a ningún backend.

</details>
