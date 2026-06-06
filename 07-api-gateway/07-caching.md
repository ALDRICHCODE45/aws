# API Gateway — Caching

## La idea base

API Gateway puede guardar las respuestas de tus endpoints en una **caché interna** (la "Gateway cache"). Cuando llega un request:

1. API Gateway busca primero en la cache.
2. Si la respuesta está cacheada y vigente → la devuelve sin tocar el backend ("cache hit").
3. Si no está o expiró → llama al backend, guarda la respuesta en cache, y la devuelve ("cache miss").

> Analogía: un mozo que anota tu pedido favorito. La próxima vez que entrás te lo trae sin pasar por la cocina. Solo va a la cocina si pedís algo nuevo o si pasó mucho tiempo desde la última vez.

## Por qué importa

1. **Menos llamadas al backend** → menos costo de Lambda, menos carga en RDS/DynamoDB/etc.
2. **Menos latencia** → la cache devuelve en milisegundos.
3. **Más throughput** → API Gateway puede servir más RPS sin escalar el backend.

## Características clave

### Se define POR STAGE (etapa)

**Esto es CRÍTICO**: la cache se habilita en un **stage** específico (`dev`, `prod`, etc.), no a nivel API global.

Consecuencia práctica:
- Podés tener cache habilitada en `prod` y deshabilitada en `dev`.
- Cada stage tiene SU propia cache, independiente de los otros.
- La cache es CARA → casi siempre solo tiene sentido en `prod`.

### TTL (Time To Live)

| Valor       | Comportamiento                                          |
| ----------- | ------------------------------------------------------- |
| **Default** | **300 segundos (5 minutos)**                            |
| **Mínimo**  | **0 segundos** → equivale a **cache desactivada**       |
| **Máximo**  | **3600 segundos (1 hora)**                              |

> **Trampa**: TTL = 0 NO significa "cache infinita", significa "no cachear". Si querés desactivar la cache de un endpoint específico, ponele TTL = 0.

### Capacidad

| Rango                                           |
| ----------------------------------------------- |
| **0.5 GB (mínimo) hasta 237 GB (máximo)**       |

Elegís el tamaño al habilitar la cache en el stage. **Más cache = más caro**.

### Cifrado (opcional)

Podés activar cifrado en reposo de los datos cacheados. Útil si la cache contiene info sensible (PII, tokens, etc.).

### Override por método

La config de cache es a nivel stage, **pero podés sobrescribirla por método**:

- Deshabilitar cache en un método específico (TTL = 0).
- Cambiar el TTL para un método (ej: GET /products con TTL=600, GET /users con TTL=60).
- Definir qué partes del request son **cache key** (ver siguiente sección).

## Cache Keys — qué hace que dos requests "sean iguales"

API Gateway necesita decidir si dos requests pueden compartir la misma respuesta cacheada. Para eso usa una **cache key** compuesta por:

- Por default: el **path** del request.
- Opcionalmente podés agregar: **query string parameters**, **headers**, **stage variables**.

Ejemplo: si marcás `?userId` como cache key, entonces:
- `GET /orders?userId=1` y `GET /orders?userId=2` se cachean por separado.
- Si NO lo marcás, los dos comparten la misma cache → BUG grave (el user 2 ve los pedidos del user 1).

> Esto es sutil pero CRÍTICO: si tu endpoint devuelve data distinta según query/headers, **tenés que incluir esos campos en la cache key**.

## Cuándo NO usar cache

- **Datos que cambian seguido** (ej: precio de criptomonedas en tiempo real).
- **Datos personalizados por usuario** sin proper cache key (riesgo de leakage).
- **POST/PUT/DELETE** que modifican estado (la cache solo tiene sentido en GET).
- **Dev/Test** → cara para lo poco que aporta.

---

# Invalidación de Cache

## El problema

Tu TTL es 300s. Pero acabás de actualizar un producto en la DB. La respuesta cacheada está VIEJA y los clientes la van a ver durante 5 minutos. **Necesitás borrar la cache YA**.

A eso se le llama **invalidar la cache**.

## Las 2 formas de invalidar

### Forma 1 — Flush total (toda la cache del stage)

Desde la consola, CLI o API:

```bash
aws apigateway flush-stage-cache --rest-api-id xxx --stage-name prod
```

Vacía TODA la cache del stage de golpe. Útil después de un deploy grande o un cambio masivo de datos.

### Forma 2 — Por request, con header

Un cliente puede invalidar una entrada específica mandando este header en su request:

```http
Cache-Control: max-age=0
```

Cuando API Gateway lo recibe:

1. **Ignora** la respuesta cacheada (si existe).
2. **Llama al backend** para traer data fresca.
3. **Reemplaza** la entrada vieja con la nueva respuesta.

> Analogía: vas al kiosko, el kiosquero te quiere dar el diario del lunes guardado. Vos le decís "dame el de hoy". Va a buscarlo Y se queda con ese para los próximos clientes.

## ⚠️ La TRAMPA DE EXAMEN (la más pedida del tema cache)

**Por DEFAULT, cualquier cliente puede mandar `Cache-Control: max-age=0` y forzar invalidación**. Esto es un problema:

- Cliente malicioso manda el header en cada request → cache nunca sirve → todos los requests van al backend → ataque tipo **cache busting** que tira tu backend.

### Solución: política IAM `execute-api:InvalidateCache`

Restringís quién puede invalidar la cache con una policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["execute-api:InvalidateCache"],
      "Resource": ["arn:aws:execute-api:region:account-id:api-id/stage-name/GET/resource-path"]
    }
  ]
}
```

**Lectura**: solo los usuarios/roles con esta policy pueden mandar `Cache-Control: max-age=0` y forzar invalidación de la cache para ese endpoint.

### Las 3 opciones de configuración en el stage

En la consola, cuando habilitás cache, configurás el comportamiento ante el header `Cache-Control: max-age=0`:

| Opción                                          | Quién puede invalidar              | Qué pasa si un cliente NO autorizado manda el header |
| ----------------------------------------------- | ---------------------------------- | ---------------------------------------------------- |
| **Require authorization (recomendado)**         | Solo con policy `InvalidateCache`  | API Gateway rechaza con **403** o lo **ignora**      |
| **No authorization required (default histórico)** | **Cualquiera** (PELIGROSO)         | El header siempre se respeta                         |

Y dentro de "Require authorization", una sub-opción:

- **Fail with 403** → rechazar el request entero.
- **Ignore + warning header** → procesar el request normal pero loggear advertencia.

## Resumen mental — cuándo se invalida la cache

| Causa                                              | Quién lo hace                          |
| -------------------------------------------------- | -------------------------------------- |
| Expira el TTL                                      | Automático                             |
| Flush manual del stage                             | Admin desde consola/CLI                |
| Header `Cache-Control: max-age=0`                  | Cliente (si tiene permiso `InvalidateCache`) |
| Update del stage que modifica config de cache      | Automático al re-deploy                |

## Trampas de examen

1. **Cache se define por STAGE**, no por API ni por método. Override por método existe pero parte de la config del stage.
2. **TTL default = 300s, mínimo 0s (desactivada), máximo 3600s (1 hora)**.
3. **TTL = 0 NO es cache infinita, es cache DESACTIVADA**.
4. **Capacidad: 0.5 GB a 237 GB**.
5. **Cache key**: por default es el path. Si tu endpoint devuelve data distinta según query/headers, **agregá esos campos a la cache key** o vas a tener data cruzada entre usuarios.
6. **Invalidación**: 2 formas → flush total del stage, o header `Cache-Control: max-age=0` por request.
7. **Por default cualquier cliente puede invalidar con el header**. Si querés restringir, configurás IAM policy con action `execute-api:InvalidateCache`.
8. **La acción IAM exacta es `execute-api:InvalidateCache`**. Memorizala — la preguntan literal.
9. **Cifrado opcional** en reposo.
10. **Solo tiene sentido en prod** — es cara, en dev/test casi siempre la desactivás.

## Auto-test

1. ¿A qué nivel se habilita la cache: API, stage o método?
2. ¿Cuál es el TTL default, mínimo y máximo?
3. ¿Qué pasa si configurás TTL = 0?
4. ¿Cuál es el rango de capacidad de la cache?
5. Tu endpoint `GET /products?category=X` devuelve productos distintos según `category`. Si NO ajustás nada, ¿qué problema vas a tener con la cache?
6. ¿Cuáles son las 2 formas de invalidar la cache?
7. ¿Qué header manda un cliente para invalidar una entrada de cache?
8. Si NO configurás ninguna policy IAM en cache, ¿quién puede invalidar?
9. ¿Cuál es la acción IAM exacta para permitir invalidación selectiva?
10. ¿Por qué la cache "casi siempre" solo tiene sentido en producción?

<details>
<summary>Respuestas</summary>

1. **Stage**. La cache se define por etapa (`dev`, `prod`, etc.). Podés override por método pero parte siempre de la config del stage.
2. **Default 300s, mínimo 0s, máximo 3600s (1 hora)**.
3. **TTL = 0 desactiva la cache** para ese stage o método. NO es cache infinita.
4. **0.5 GB a 237 GB**.
5. La cache key default es solo el path. Sin agregar `category` a la cache key, **todos los `GET /products?category=X` comparten la misma respuesta cacheada** → un usuario que pide `category=zapatos` puede recibir productos de `category=libros`. Hay que agregar `category` a la cache key.
6. **(a)** Flush total del stage desde consola/CLI (`flush-stage-cache`). **(b)** Header `Cache-Control: max-age=0` en un request individual.
7. **`Cache-Control: max-age=0`**.
8. **Cualquier cliente** puede invalidar mandando el header. Es un riesgo de "cache busting" — un atacante puede tirar abajo tu cache y saturar el backend.
9. **`execute-api:InvalidateCache`**. Se aplica vía IAM policy en el usuario/rol del cliente.
10. La cache es **cara** (pagás por GB y por tiempo encendido). En dev/test el tráfico es bajo y la data cambia seguido, así que el costo no se justifica. En prod, donde tenés alto tráfico repetido sobre los mismos endpoints, ahorra mucho en backend.

</details>
