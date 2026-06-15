# 🚧 PENDIENTE — Retomar acá la próxima sesión

> Última actualización: 2026-06-15 (sesión interrumpida por cambio de compu).

## Estado del bloque API Gateway

| Apunte | Estado |
| ------ | ------ |
| 01-fundamentals | ✅ |
| 02-stages-and-deployments | ✅ |
| 03-stage-variables | ✅ |
| 04-canary-deployments | ✅ |
| 05-integration-types | ✅ |
| 06-openapi-and-validation | ✅ |
| 07-caching | ✅ |
| 08-usage-plans-and-api-keys | ✅ |
| **09-logs-metrics-and-xray** | 🟡 **EXPLICACIÓN HECHA, FALTA APUNTE** |
| **10-cors** | 🟡 **EXPLICACIÓN HECHA, FALTA APUNTE** |

## 🎯 Qué tengo que hacer cuando volvamos

### 1. Pregunta pendiente de Logs/X-Ray (responder primero)

> Tu equipo se queja de que la API "está lenta". Tenés un endpoint `POST /checkout` que pasa por API Gateway → Lambda → DynamoDB → Stripe (HTTP externo). Latency promedio: 2.5 segundos. Tu jefe quiere saber **DÓNDE exactamente está el cuello de botella**.
>
> - **A)** Aumentar el log level a DEBUG en CloudWatch Logs del stage de producción y revisar los logs uno por uno.
> - **B)** Mirar la métrica `Latency` y la métrica `IntegrationLatency` en CloudWatch Metrics y comparar.
> - **C)** Habilitar X-Ray en API Gateway y en Lambda (con SDK), y mirar el service map para ver el timing por servicio.
> - **D)** Habilitar Access Logs en formato JSON y procesarlos con Athena.
>
> ¿Cuál es la correcta? Justificá descartando las otras 3. Hay una **parcialmente útil pero no la mejor**.

**Cuando responda → armar apunte `09-logs-metrics-and-xray.md`** con todo lo explicado (ver sección abajo).

### 2. Pregunta pendiente de CORS (responder segundo)

> Tu frontend en `https://app.midominio.com` (S3) llama a tu API en `https://api.midominio.com` (API Gateway → Lambda). El endpoint es `POST /orders` con `Content-Type: application/json` y un header custom `X-Auth-Token`. Habilitaste "Enable CORS" en la consola de API Gateway sobre el recurso `/orders` y redeployaste el stage `prod`. Pero al probar desde el navegador te sale:
>
> *"CORS error: Response to preflight request doesn't pass access control check"*
>
> ¿Cuáles son las **2 causas más probables** y cómo las verificás?

**Cuando responda → armar apunte `10-cors.md`** con todo lo explicado.

---

## 📚 Lo que ya le expliqué (para armar los apuntes después)

### Apunte 09 — Logs, Metrics y X-Ray (contenido a destilar)

**Concepto clave**: 3 tipos de telemetría, no son lo mismo.

| Tipo | Qué te dice | Servicio | Granularidad |
| ---- | ----------- | -------- | ------------ |
| **Metrics** | Números agregados (latencia, count, errores) | CloudWatch Metrics | Por stage/método |
| **Logs** | Eventos individuales (qué pasó en cada request) | CloudWatch Logs | Por request |
| **Traces** | Recorrido de UN request entre servicios | X-Ray | Por request, cross-service |

Analogía: Metrics = velocímetro / Logs = bitácora / Traces = GPS con historial.

**CloudWatch Metrics (automático, sin habilitar):**
- `CacheHitCount` / `CacheMissCount` — eficiencia de la cache
- `Count` — tráfico total
- `Latency` — tiempo TOTAL (cliente → respuesta)
- `IntegrationLatency` — tiempo SOLO del backend
- `4XXError` — errores cliente
- `5XXError` — errores backend / API Gateway

**⚠️ Trampa crítica: Latency vs IntegrationLatency**
- `Latency` = tiempo total (incluye API Gateway + backend)
- `IntegrationLatency` = tiempo SOLO del backend
- Si `Latency ≈ IntegrationLatency` → el backend es el cuello.
- Si `Latency >> IntegrationLatency` → API Gateway (cache, throttling, auth, etc.).
- Memorizar los nombres EXACTOS — caen literal.

**Log levels de CloudWatch Logs:**

| Nivel | Qué loggea | Cuándo | Costo |
| ----- | ---------- | ------ | ----- |
| OFF (default) | Nada | Apagado | $0 |
| ERROR | Solo errores (4XX, 5XX) | Producción normal | Bajo |
| INFO | Errores + info general | Producción con debugging eventual | Medio |
| DEBUG | TODO: headers, body, response | Debugging temporal en dev | **ALTO** |

**⚠️ Trampa DEBUG en producción** — NUNCA permanente:
1. Costo: 100x más volumen.
2. Seguridad: loggea PII, tokens, passwords, tarjetas.
3. Performance: overhead I/O.

**Dos tipos de logs (NO está en la diapo pero entra):**

| Tipo | Qué loggea | Mnemónico |
| ---- | ---------- | --------- |
| **Execution Logs** | Lo que pasa DENTRO de API Gateway | "¿qué pasó adentro?" |
| **Access Logs** | Una línea por request (Apache/Nginx style) | "¿quién entró?" |

Se habilitan independientes. Mismo patrón stage por default, override por método.

**X-Ray — trazabilidad distribuida:**

Conceptos:
- **Trace** = recorrido completo cross-service
- **Segment** = lo que hizo UN servicio
- **Subsegment** = operación interna (query DynamoDB, llamada HTTP)
- **Sampling** = no traza todos los requests por costo

**⚠️ Para que X-Ray funcione end-to-end**:
1. Habilitar en API Gateway (stage settings)
2. Habilitar Active Tracing en Lambda (function config)
3. Usar AWS X-Ray SDK en código Lambda (para trazas internas)

Por eso la diapo dice "API Gateway + Lambda = imagen completa".

**Tabla mental "qué servicio uso":**

| Necesidad | Servicio |
| --------- | -------- |
| "Cuántas requests" / "latencia promedio" | CloudWatch Metrics |
| "Qué pasó EXACTAMENTE en este request fallido" | CloudWatch Logs (Execution) |
| "Quién entró y desde qué IP" | CloudWatch Logs (Access) |
| "Por qué tarda 3s pasando por 5 servicios" | X-Ray |
| "Comparar performance API GW vs backend" | Latency vs IntegrationLatency |
| "Detectar throttling / 429s" | CloudWatch Metrics |
| "Alertar cuando 5XX > 1%" | CloudWatch Alarms |

### Apunte 10 — CORS (contenido a destilar)

**Concepto que le costó al usuario**: pensaba que CORS = "poner orígenes permitidos en la API". NO está mal, está incompleto.

**Lo que CORS realmente es**: protocolo de 2 pasos entre navegador y servidor, NO una config "ponés y listo".

**Por qué EXISTE CORS — el problema real:**

Sin CORS, un sitio malicioso podría hacer `fetch('https://www.mibanco.com/api/transferir')` desde el navegador del usuario logueado y mandar las cookies de sesión → robo automático.

**Same-Origin Policy (SOP)**: por default, una página solo puede hacer AJAX a su mismo origen.

**Mismo origen** = protocolo + dominio + puerto idénticos.

| URL A | URL B | ¿Mismo origen? |
| ----- | ----- | -------------- |
| `https://www.example.com` | `https://www.example.com/api` | ✅ SÍ |
| `https://www.example.com` | `https://api.example.com` | ❌ NO (subdominio) |
| `https://www.example.com` | `http://www.example.com` | ❌ NO (protocolo) |
| `https://www.example.com` | `https://www.example.com:8080` | ❌ NO (puerto) |

**CORS = mecanismo para AFLOJAR la SOP de forma controlada.**

**⚠️ CONCEPTO CLAVE QUE SE PIERDE**: CORS lo aplica **EL NAVEGADOR**, no el servidor. El servidor solo manda headers diciendo "yo permito esto", el navegador decide si bloquear o dejar pasar la respuesta al JS.

**Flujo de 2 pasos (preflighted):**

1. **Preflight (OPTIONS)** — el navegador automáticamente pregunta:
   ```
   OPTIONS /
   Host: api.example.com
   Origin: https://www.example.com
   Access-Control-Request-Method: PUT
   ```
2. **Respuesta preflight** — API Gateway responde:
   ```
   Access-Control-Allow-Origin: https://www.example.com
   Access-Control-Allow-Methods: GET, PUT, DELETE
   ```
3. **Request real** — recién ahora el GET/PUT/DELETE real.

> Analogía: como cuando vas al boliche con un amigo de afuera y antes de entrar le preguntás al de la puerta "¿podemos entrar con remera y zapatillas?". Si dice sí, recién ahí entrás.

**Simple vs Preflighted requests:**

| Tipo | Cuándo | ¿OPTIONS? |
| ---- | ------ | --------- |
| Simple | GET, HEAD, o POST con form-urlencoded/text/plain/multipart | ❌ NO |
| Preflighted | PUT, DELETE, PATCH, POST con `application/json`, o headers custom | ✅ SÍ |

> Las APIs modernas usan JSON → casi siempre hay preflight.

**Headers CORS principales:**

| Header (response) | Para qué |
| ----------------- | -------- |
| `Access-Control-Allow-Origin` | Qué origen permitido (`*` o dominio exacto) |
| `Access-Control-Allow-Methods` | Métodos HTTP permitidos |
| `Access-Control-Allow-Headers` | Headers custom permitidos |
| `Access-Control-Allow-Credentials` | Si se mandan cookies (true/false) |
| `Access-Control-Max-Age` | Cuántos seg cachear el preflight |

**⚠️ Trampa de seguridad**: `Access-Control-Allow-Origin: *` + `Access-Control-Allow-Credentials: true` → el navegador rechaza la combinación. No podés permitir cookies a "cualquier origen".

**Lo que API Gateway hace por vos:**

Botón "Enable CORS" automáticamente:
1. Crea método OPTIONS en el recurso.
2. Configura headers `Access-Control-Allow-*`.
3. Te deja editarlos.

**⚠️ Trampas pedidísimas en el examen:**
- Olvidar habilitar CORS en TODOS los recursos (cada path necesita su OPTIONS).
- Olvidar REDEPLOYAR el stage después → cambios NO surten efecto sin deploy (trampa más cargosa del bloque).
- Lambda devuelve 500 sin headers CORS → el navegador igual bloquea.

**Tabla mental quién hace qué:**

| Quién | Qué hace |
| ----- | -------- |
| Navegador | Detecta cross-origin, manda preflight, bloquea si headers no permiten |
| API Gateway | Responde el preflight automáticamente |
| Backend (Lambda) | NO se entera de CORS — API GW lo maneja antes |
| Vos | Habilitás CORS en consola + redeploy |
