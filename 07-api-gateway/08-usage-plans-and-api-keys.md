# API Gateway — Usage Plans y API Keys

## El problema que resuelven

Imaginá que armaste una API y la querés **monetizar** — venderla a terceros como servicio. Te aparecen 3 problemas:

1. **¿Cómo identificás a cada cliente que te consume la API?**
2. **¿Cómo cobrás distinto a cada uno?** (no es lo mismo un cliente que hace 100 req/mes que uno que hace 10 millones)
3. **¿Cómo evitás que un cliente del plan barato te haga 1 millón de requests y te funda la cuenta de AWS?**

Para esto existen **API Keys** + **Usage Plans**. Son dos cosas distintas que trabajan juntas.

---

## API Keys — el "DNI" del cliente

Una **API Key** es un string alfanumérico largo que identifica a un cliente:

```
WBjHxNtoAb4WPKBC7cGm64CBiblb24b4jt8jJHo9
```

**Flujo:**

1. Vos generás la API Key desde API Gateway.
2. Se la entregás al cliente.
3. El cliente la manda en cada request, en el header **`x-api-key`**:
   ```http
   GET /weather HTTP/1.1
   x-api-key: WBjHxNtoAb4WPKBC7cGm64CBiblb24b4jt8jJHo9
   ```
4. API Gateway lee el header, identifica al cliente, y aplica los límites del Usage Plan asociado.

> Analogía: la API Key es el **carnet de socio del gimnasio**. Te identifica al entrar, pero no prueba que seas vos (alguien te puede robar la tarjeta). Para probar identidad de verdad necesitás auth real.

### ⚠️ Trampa CRÍTICA de examen

**La API Key NO es autenticación. Solo identifica.**

| Para…                             | Usás…                           |
| --------------------------------- | ------------------------------- |
| **Identificar y medir consumo**   | API Key                         |
| **Autenticar (probar quién sos)** | Cognito, IAM, Lambda Authorizer |

Si en el examen ves "se necesita autenticar usuarios de la API" y la opción dice "usar API Keys" → es **TRAMPA**. La respuesta correcta va a ser Cognito o Lambda Authorizer.

---

## Usage Plan — las reglas del juego

Una **API Key sola no hace nada** — es solo un string. Lo que define **qué puede hacer ese cliente** es el **Usage Plan**.

Un Usage Plan es un **contrato** que dice:

- A qué **stages** y **métodos** puede acceder.
- **Throttling**: cuántas requests por segundo puede hacer.
- **Quota**: cuántas requests por mes/día/semana puede hacer en total.

### Ejemplo concreto — API de clima con 3 planes

| Plan       | Throttling | Quota mensual     | Precio |
| ---------- | ---------- | ----------------- | ------ |
| Free       | 10 req/s   | 10.000 req/mes    | $0     |
| Pro        | 100 req/s  | 1.000.000 req/mes | $50    |
| Enterprise | 1000 req/s | Sin límite        | $500   |

Cada fila es un **Usage Plan**. Cuando un cliente se suscribe a "Pro", le generás una API Key y la **asociás** a ese Usage Plan.

---

## Throttling vs Quota — NO son lo mismo

Esto cae seguro en el examen:

| Concepto                          | Qué limita                  | Ejemplo             | Cuando se supera                    |
| --------------------------------- | --------------------------- | ------------------- | ----------------------------------- |
| **Throttling (estrangulamiento)** | **Velocidad** (req/seg)     | "máximo 100 req/s"  | 429 al toque                        |
| **Quota (cuota)**                 | **Volumen total** (req/mes) | "máximo 1M req/mes" | 429 hasta que se renueve el período |

> **Analogía**: el **throttling** es el límite de velocidad de la ruta (no podés ir a más de 110 km/h en ningún momento). La **quota** es el kilometraje del alquiler (tenés 2000 km totales para todo el viaje).

### Detalle fino — Rate vs Burst (token bucket)

El throttling de API Gateway usa el algoritmo **token bucket**, con 2 valores:

- **Rate**: el promedio sostenido (ej: 100 req/s).
- **Burst**: ráfaga corta permitida por encima del rate (ej: 200 burst).

Si el cliente está quieto un rato y de repente manda 200 requests, le entran todas (consume el "balde" de burst). Pero si sigue mandando, queda limitado al rate de 100/s.

No memorices números — solo tené presente que **rate ≠ burst** y que se permiten ráfagas cortas.

---

## Cómo se conecta todo — el flujo mental

```
Cliente
   │
   │ HTTP request + header x-api-key: ABC123...
   ▼
API Gateway
   │
   │ 1. ¿Existe esta API Key? ──── NO ──► 403 Forbidden
   │           SI
   │           ▼
   │ 2. ¿A qué Usage Plan está asociada?
   │           ▼
   │ 3. ¿Ya superó la quota del período? ──── SI ──► 429 Too Many Requests
   │           NO
   │           ▼
   │ 4. ¿Está superando el throttling (req/s)? ── SI ──► 429 Too Many Requests
   │           NO
   │           ▼
   │ 5. ¿Tiene permiso para este stage/método? ── NO ──► 403 Forbidden
   │           SI
   │           ▼
   │ 6. Pasa la request al backend (Lambda, etc.)
```

---

## Las 3 capas de throttling de API Gateway

API Gateway aplica throttling en **3 niveles**, en cascada de afuera hacia adentro:

| Nivel                             | Quién lo configura                      | Para qué sirve                             |
| --------------------------------- | --------------------------------------- | ------------------------------------------ |
| **1. Cuenta AWS (account level)** | AWS (default 10.000 req/s, 5.000 burst) | Techo global de toda tu cuenta             |
| **2. Stage / Method level**       | Vos                                     | Proteger un stage o un endpoint específico |
| **3. Usage Plan (per API Key)**   | Vos                                     | Limitar a cada cliente individual          |

**Regla**: gana el **más restrictivo**. Si tu cuenta soporta 10.000 req/s pero el Usage Plan dice 100 req/s, al cliente le aplican 100.

---

## Códigos HTTP — quién devuelve qué

| Código                    | Cuándo                                                       |
| ------------------------- | ------------------------------------------------------------ |
| **200 OK**                | Pasó todos los chequeos                                      |
| **403 Forbidden**         | API Key inválida, inexistente, o sin permiso al stage/método |
| **429 Too Many Requests** | Superó throttling (req/s) o quota (total del período)        |

> No confundas: **403 = no podés** (key mala o sin acceso). **429 = pasaste el límite** (key buena pero ya consumiste de más).

---

---

## ¿API Keys o Lambda Authorizer? — árbol de decisión

Esta es **LA confusión más cara del examen**. Las API Keys + Usage Plans NO sirven para "límite por usuario individual". Sirven para **categorías de clientes comerciales**.

### Regla mental

```
¿Para qué necesitás los límites?
│
├── Para tiers comerciales (Free / Pro / Enterprise)
│   → API Keys + Usage Plans ✅
│
├── Para usuarios individuales de una app
│   → Lambda Authorizer + rate limiting custom
│     (DynamoDB con TTL, ElastiCache/Redis, etc.) ✅
│
├── Para solo autenticar usuarios de app móvil/web
│   → Cognito User Pools (sin API Keys) ✅
│
└── Para auth con IdP propio (Auth0, Okta, JWT custom)
    → Lambda Authorizer ✅
```

### Por qué API Keys NO funciona "por usuario"

Pensalo así: cada usuario necesitaría su propia API Key + su propio Usage Plan asociado. Con **1000 usuarios → 1000 API Keys + 1000 Usage Plans**. Inviable y caro. Por eso se usan solo para **clientes B2B comerciales**, donde tenés decenas o cientos como mucho.

### Tabla de palabras clave en el enunciado

| Si la pregunta dice…            | Respuesta NO es API Keys |
| ------------------------------- | ------------------------ |
| "millones de usuarios"          | ❌                       |
| "por usuario individual"        | ❌                       |
| "usuarios de una app móvil/web" | ❌                       |
| "autenticar usuarios finales"   | ❌                       |
| "end users"                     | ❌                       |

| Si la pregunta dice…                          | Respuesta SÍ es API Keys |
| --------------------------------------------- | ------------------------ |
| "clientes comerciales" / "B2B"                | ✅                       |
| "monetizar / vender la API"                   | ✅                       |
| "tiers / planes de uso" (Free/Pro/Enterprise) | ✅                       |
| "third-party developers / partners"           | ✅                       |
| "diferentes niveles de servicio"              | ✅                       |

### Aclaración importante: Lambda Authorizer NO es "solo para autorizar"

A pesar del nombre, **hace AMBAS**: autentica Y autoriza. Y además pasa contexto del usuario al backend, lo que te permite implementar rate limiting per-user con DynamoDB o Redis.

| Función                                   | Lambda Authorizer la hace? |
| ----------------------------------------- | -------------------------- |
| Validar token / autenticar                | ✅                         |
| Devolver IAM policy (autorizar)           | ✅                         |
| Pasar contexto del usuario al backend     | ✅                         |
| Habilitar rate limiting per-user (custom) | ✅                         |

---

## Trampas de examen

1. **API Key ≠ autenticación.** Solo identifica y mide consumo. Para auth real → Cognito / IAM / Lambda Authorizer.
2. **API Key sola no hace nada.** Necesita estar asociada a un **Usage Plan** para que aplique límites.
3. **Header obligatorio: `x-api-key`** (en minúsculas, con guión).
4. **Throttling = req/s** (velocidad). **Quota = req/mes/día** (volumen total).
5. **Cuando se supera el límite → HTTP 429 Too Many Requests** (NO es 403).
6. **403 Forbidden** = la key no existe o no tiene permiso al stage/método.
7. Throttling se aplica en **3 capas**: cuenta AWS → stage/método → usage plan. Gana el más restrictivo.
8. Un Usage Plan se asocia a **uno o más stages**, no a una API entera de forma global.
9. Una API Key puede estar asociada a **varios Usage Plans** (uno por API/stage).
10. **Rate ≠ Burst**: el throttling permite ráfagas cortas vía token bucket.
11. **API Keys + Usage Plans son para tiers comerciales (Free/Pro/Enterprise), NO para usuarios individuales.** Para límite por usuario → Lambda Authorizer + DynamoDB/Redis custom.
12. **Lambda Authorizer hace AMBAS: autentica Y autoriza.** El nombre engaña — también valida tokens, no solo permisos.

---

## Auto-test

1. ¿Qué es una API Key y para qué sirve?
2. ¿La API Key autentica al usuario? Si no, ¿qué servicios de AWS sí lo hacen?
3. ¿En qué header se manda la API Key?
4. ¿Cuál es la diferencia entre throttling y quota?
5. Tenés un Usage Plan "Free" con throttling 100 req/s y quota 10.000 req/mes. Un cliente manda 500 requests en 1 segundo. ¿Qué pasa con cada una y qué código HTTP devuelve?
6. ¿Qué pasa si superás la quota mensual? ¿Y cuándo se "renueva"?
7. ¿Cuáles son las 3 capas de throttling en API Gateway? ¿Cuál gana cuando hay conflicto?
8. ¿Qué diferencia hay entre el código 403 y el 429 en API Gateway?
9. Si un cliente manda un request SIN el header `x-api-key` a un endpoint que requiere API Key, ¿qué código recibe?
10. Pregunta tramposa: un cliente compra el plan "Enterprise" (1000 req/s) pero tu cuenta AWS está al tope (10.000 req/s) y otros clientes ya consumen 9.500 req/s. ¿Qué le pasa al Enterprise si intenta meter sus 1000 req/s?
11. **Pregunta tramposa avanzada**: tu app móvil tiene 500.000 usuarios. Necesitás autenticarlos Y aplicar un límite de 100 req/min por usuario. ¿Cuál es la arquitectura correcta y por qué descartás Cognito + API Keys con Usage Plans?

<details>
<summary>Respuestas</summary>

1. Una **API Key** es un string alfanumérico que **identifica a un cliente** de tu API. Sirve para medir su consumo y aplicar los límites del Usage Plan asociado. NO autentica.
2. **No autentica**, solo identifica. Para autenticación real → **Cognito User Pools**, **IAM**, o **Lambda Authorizer** (custom authorizer).
3. En el header **`x-api-key`** (minúsculas, con guión).
4. **Throttling** limita la **velocidad** (req/s, protege de picos). **Quota** limita el **volumen total** (req/mes o día, protege el modelo de negocio).
5. Las primeras **100 requests pasan** (200 OK), las **400 restantes se rechazan con 429 Too Many Requests**. La quota mensual baja de 10.000 a 9.900 (solo cuentan las que pasaron).
6. Si superás la quota → todas las siguientes requests devuelven **429** hasta que se cumple el período (al inicio del próximo mes, día o semana según cómo configuraste el Usage Plan).
7. (1) **Cuenta AWS** (10.000 req/s, 5.000 burst por default), (2) **Stage / Method**, (3) **Usage Plan (per API Key)**. Gana **el más restrictivo**.
8. **403 Forbidden** = problema de **identidad o permisos** (API Key inválida, inexistente, o sin acceso al stage/método). **429 Too Many Requests** = la key es válida pero **superó algún límite** (throttling o quota).
9. **403 Forbidden** — falta la key y el endpoint la requiere. No es 401 (API Gateway no usa 401 para este caso porque la key no es auth, es identificación).
10. Le devuelve **429 Too Many Requests** aunque su Usage Plan permita 1000 req/s. La **cuenta AWS está al límite** y esa es la capa más alta de throttling. La regla del "más restrictivo gana" no aplica acá porque la cuenta está saturada por OTROS clientes que ya consumieron la capacidad. Es un problema de capacidad compartida — se soluciona pidiéndole a AWS un límite de cuenta más alto (service quota increase).
11. La arquitectura correcta es **Lambda Authorizer + rate limiting custom (DynamoDB con TTL o ElastiCache/Redis)**. Descartás "Cognito + API Keys con Usage Plans" porque los Usage Plans aplican límites **a la API Key**, no al usuario. Con 500.000 usuarios necesitarías 500.000 API Keys y 500.000 Usage Plans — inviable. Cognito por sí solo autentica pero no aplica rate limiting per-user. La solución real es: Lambda Authorizer valida el JWT (de Cognito o propio), extrae el userId, lo pasa como context al backend, y el backend (o el propio authorizer) hace rate limiting consultando un counter en DynamoDB/Redis indexado por userId.

</details>
