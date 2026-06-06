# API Gateway — OpenAPI & Request Validation

## La idea base

**OpenAPI** (antes Swagger) es el estándar de la industria para describir APIs REST en un archivo JSON o YAML. API Gateway lo soporta nativamente: podés **importar** un OpenAPI para crear tu API entera, o **exportar** tu API existente.

Encima, API Gateway agrega **extensiones propietarias** (todas con prefijo `x-amazon-apigateway-...`) para configurar cosas específicas de AWS: integraciones, autorizadores, validadores, etc.

> Analogía: OpenAPI es el "plano arquitectónico" de tu API. AWS le agrega anotaciones propias para que el plano describa también cómo se construye en API Gateway.

## ¿Para qué sirve OpenAPI en API Gateway?

Tres usos grandes:

### 1. Importar — la API como código

Definís todo tu API en un YAML/JSON: paths, methods, integraciones, models, validators, auth. Hacés **import** y API Gateway crea TODO en un solo paso.

Es Infrastructure-as-Code para API Gateway **sin necesidad de CloudFormation**. Bueno para versionar la API en Git.

### 2. Exportar — documentación y portabilidad

Tomás una API ya construida en API Gateway y la exportás en JSON o YAML. Sirve para:

- Documentación (Swagger UI, Redoc, Postman).
- Migrar/clonar la API a otra cuenta o región.
- Generar diff/changelog entre versiones.

### 3. Generar SDKs

Desde la consola o CLI, generás un **SDK cliente** para llamar a tu API:

- JavaScript, iOS (Objective-C/Swift), Android (Java), Java SE, Ruby.
- El SDK incluye auth, retry, signing si hace falta, y los métodos tipados según tu OpenAPI.

> Beneficio: los developers cliente NO escriben el HTTP a mano. Llaman `client.createOrder({...})` con tipos.

## Formato: JSON o YAML, ambos válidos

API Gateway soporta los dos. La doc oficial muestra ejemplos en los dos formatos. Elegís según preferencia del equipo.

OpenAPI tiene dos versiones soportadas:

- **OpenAPI 2.0** (antes "Swagger 2.0")
- **OpenAPI 3.0**

Las extensiones AWS funcionan en ambos.

---

# Request Validation

## La idea base

Antes de que el request llegue al backend (Lambda, HTTP, lo que sea), **API Gateway puede validar el request** y rechazarlo con `400 Bad Request` si está mal formado.

> Analogía: el portero del edificio revisa tu credencial ANTES de dejarte subir al ascensor. Si la credencial está vencida, no llamás al ascensor (no gastás backend).

## ¿Por qué importa?

1. **Ahorra invocaciones de backend** (menos costo de Lambda, menos carga en HTTP backends).
2. **Falla rápido** — el cliente recibe el error en milisegundos sin esperar al backend.
3. **Centraliza validación básica** — no tenés que repetirla en cada Lambda.

## ¿Qué valida exactamente?

API Gateway valida **DOS cosas**, nada más:

### A) Request parameters

Que los parámetros marcados como **required** estén presentes y no vacíos en:

- **URI path** (`/users/{id}` → `{id}` no puede faltar).
- **Query string** (`?status=active` → `status` no puede estar vacío si es required).
- **Headers** (`X-Api-Version` → idem).

> NO valida el VALOR del parámetro, solo que esté presente y no vacío. Para validar formato/regex, usás un Model con JSON Schema en el body, o un Lambda Authorizer.

### B) Request body (payload)

Que el body del request matchee un **Model** (JSON Schema) que vos definís.

El Model puede chequear:

- Campos requeridos (`required: ["email", "name"]`).
- Tipos (`type: string`, `type: integer`).
- Formato (`format: email`, `format: date-time`).
- Restricciones (`minLength`, `maxLength`, `minimum`, `maximum`, `pattern`).
- Estructura anidada de objetos.

## Lo que NO valida (trampa de examen)

- ❌ **Reglas de negocio** (ej: "el email debe existir en la DB"). Eso es lógica del backend.
- ❌ **Autorización** (eso es Authorizer, otro tema).
- ❌ **El response del backend** (solo valida el request entrante).
- ❌ **Valores específicos de query/headers** (solo presencia/no-vacío). Para eso, ponelos en el body con un Model.

## Los 3 modos de validador

Definís uno o más validators en `x-amazon-apigateway-request-validators` con dos flags booleanos:

| Validador     | `validateRequestParameters` | `validateRequestBody` | Qué valida                       |
| ------------- | --------------------------- | --------------------- | -------------------------------- |
| `params-only` | `true`                      | `false`               | Solo params (URI/query/headers)  |
| `body-only`   | `false`                     | `true`                | Solo body contra el Model        |
| `all`         | `true`                      | `true`                | Ambos                            |

Los **nombres** (`params-only`, `body-only`, `all`) son arbitrarios — los elegís vos. AWS los usa así por convención, pero podés llamarlos como quieras (`basic`, `strict`, `payload-check`, etc.).

## Aplicar el validador: API-level vs method-level

Una vez definidos los validators, los **asignás** con la extensión `x-amazon-apigateway-request-validator` (singular, OJO con la S).

### Default a nivel API (afecta a todos los métodos)

```json
{
  "openapi": "3.0.0",
  "x-amazon-apigateway-request-validators": {
    "params-only": { "validateRequestBody": false, "validateRequestParameters": true }
  },
  "x-amazon-apigateway-request-validator": "params-only"
}
```

Todos los métodos heredan `params-only`.

### Override por método

```json
{
  "paths": {
    "/validation": {
      "post": {
        "x-amazon-apigateway-request-validator": "all"
      }
    }
  }
}
```

`POST /validation` usa `all`, ignorando el `params-only` heredado de la API.

> **Regla de oro: el método siempre gana sobre la API.**

## Ejemplo completo (de la doc oficial)

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "ReqValidation Sample",
    "version": "1.0.0"
  },
  "x-amazon-apigateway-request-validators": {
    "all": {
      "validateRequestBody": true,
      "validateRequestParameters": true
    },
    "params-only": {
      "validateRequestBody": false,
      "validateRequestParameters": true
    }
  },
  "x-amazon-apigateway-request-validator": "params-only",
  "paths": {
    "/validation": {
      "post": {
        "x-amazon-apigateway-request-validator": "all"
      }
    }
  }
}
```

**Lectura:**

1. Defino DOS validators: `all` y `params-only`.
2. Default API: `params-only`.
3. `POST /validation` override a `all`.
4. Cualquier otro método de la API valida solo parámetros.

## Models (JSON Schema)

Para validar el body, definís un **Model** y lo asociás al método. El Model es un JSON Schema estándar.

Ejemplo:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "title": "OrderModel",
  "type": "object",
  "required": ["item", "quantity"],
  "properties": {
    "item": { "type": "string", "minLength": 1 },
    "quantity": { "type": "integer", "minimum": 1, "maximum": 100 },
    "email": { "type": "string", "format": "email" }
  }
}
```

Si el cliente manda `{"item": "pizza", "quantity": 0}`, el validador `body-only` o `all` rechaza con `400` ANTES de llegar al backend.

## Qué pasa cuando falla la validación

API Gateway responde inmediatamente con:

- **HTTP Status**: `400 Bad Request`
- **Body**: mensaje describiendo qué falló (campo faltante, tipo incorrecto, etc.).
- **El backend NO se invoca** (no se cobra Lambda, no se gasta capacidad HTTP).

## Trampas de examen

1. **OpenAPI = import/export + SDK generation**. Soporta JSON y YAML, OpenAPI 2.0 y 3.0.
2. **Request Validation valida SOLO 2 cosas**: parámetros requeridos (presencia/no-vacío) y body contra JSON Schema Model.
3. **No valida lógica de negocio ni autorización**. Solo estructura.
4. **Falla con 400 Bad Request** ANTES de invocar el backend (ahorra costo).
5. **3 modos**: `params-only`, `body-only`, `all`. Los nombres son arbitrarios, vos los definís.
6. **Asignación jerárquica**: validator a nivel API se hereda, validator a nivel método sobrescribe.
7. **Extensiones AWS** todas empiezan con `x-amazon-apigateway-...`. Para validators son DOS extensiones distintas:
   - `x-amazon-apigateway-request-validators` (plural) → DEFINE los validators.
   - `x-amazon-apigateway-request-validator` (singular) → ASIGNA un validator a la API o a un método.
8. **Models** = JSON Schema. Se referencian desde el método y el validator de `body` los matchea contra el payload.

## Auto-test

1. ¿Para qué tres cosas grandes sirve OpenAPI en API Gateway?
2. ¿Qué dos cosas valida exactamente API Gateway con Request Validation?
3. Si la validación falla, ¿qué status code devuelve API Gateway y se invoca el backend?
4. ¿Cuáles son los 3 modos típicos de validator y qué diferencia tienen?
5. Tenés un validator `params-only` configurado a nivel API y `all` configurado en `POST /orders`. ¿Cuál se aplica en `POST /orders`?
6. ¿Cuál es la diferencia entre las extensiones `x-amazon-apigateway-request-validators` y `x-amazon-apigateway-request-validator`?
7. ¿Podés validar con Request Validation que el email del usuario exista en tu base de datos?
8. ¿Qué es un "Model" en API Gateway y para qué se usa en validación?

<details>
<summary>Respuestas</summary>

1. **(a)** Importar para crear la API como código, **(b)** exportar para documentación/portabilidad, **(c)** generar SDKs cliente (JS, iOS, Android, Java, Ruby).
2. **(a)** Parámetros required (URI/query/headers) presentes y no vacíos, **(b)** body matcheando un Model JSON Schema.
3. **400 Bad Request**. El backend **NO se invoca** — ahorrás costo de Lambda/HTTP.
4. **`params-only`** valida solo parámetros. **`body-only`** valida solo el body contra el Model. **`all`** valida ambos. Los nombres son arbitrarios — los elegís vos.
5. **`all`**. El método siempre gana sobre la API. La configuración a nivel método sobrescribe la heredada.
6. **`x-amazon-apigateway-request-validators`** (plural) **DEFINE** los validators con sus reglas. **`x-amazon-apigateway-request-validator`** (singular) **ASIGNA** un validator ya definido a la API o a un método.
7. **NO**. Request Validation valida solo estructura/formato. Reglas de negocio (existencia en DB, permisos, etc.) son responsabilidad del backend.
8. Un **Model** es un **JSON Schema** que describe la estructura esperada del body. Se asocia a un método y el validator de `body` matchea el payload contra el Model. Si no cumple → 400.

</details>
