# API Gateway — Fundamentos

## Qué es

Servicio **serverless totalmente gestionado** por AWS para crear, publicar, mantener, monitorear y asegurar APIs. Actúa como **"puerta de entrada"** (front door) entre los clientes (web, mobile, IoT, otros servicios) y tu backend (Lambda, EC2, ECS, servicios HTTP, AWS services).

> Analogía: API Gateway es el **recepcionista** de un edificio. El cliente llega, el recepcionista valida quién sos (auth), te limita el paso si hay mucha gente (throttling), te dirige al piso correcto (routing al backend) y te puede dar info cacheada sin molestar al de adentro.

## Por qué existe

Sin API Gateway, cada backend tendría que resolver por su cuenta: autenticación, throttling, versionado, CORS, logging, caching, validación de requests, transformación de payloads. API Gateway centraliza TODO eso en una capa.

## Características clave

- **Serverless** — no manejás servidores, AWS escala automáticamente.
- **Soporta múltiples protocolos** — REST, HTTP, WebSocket.
- **Integración nativa con Lambda** — el combo serverless por excelencia.
- **También integra con** — servicios HTTP cualquiera (público o VPC), servicios AWS (DynamoDB, S3, Step Functions, SQS, etc.), backends on-premise vía Direct Connect.
- **Versionado de APIs** — v1, v2, v3 conviviendo.
- **Manejo de stages** — dev, test, prod, cada uno con su config.
- **Seguridad** — IAM, Cognito, Lambda Authorizers, API Keys.
- **Rate limiting / throttling** — protección contra abuso.
- **Cache** — para reducir llamadas al backend.
- **Transformación de requests/responses** — modificá el payload antes de llegar al backend o antes de devolver al cliente.
- **Generación de SDKs** — para iOS, Android, JavaScript.
- **Swagger / OpenAPI** — importar/exportar definiciones.
- **CORS** — configuración integrada.

## Tipos de API Gateway

AWS ofrece **3 tipos**, y elegir el correcto es **trampa clásica de examen**:

### 1. REST API

- El "completo", con TODAS las features.
- Cache, request validation, transformations, API keys, usage plans, X-Ray, WAF.
- **Más caro** y un poco más lento que HTTP API.
- Usalo cuando necesitás features avanzadas.

### 2. HTTP API

- Versión **más nueva, más simple, más barata y más rápida** que REST API.
- **Hasta 71% más barata** que REST API.
- Soporta JWT authorizers nativos (Cognito, Auth0, etc.).
- **NO tiene**: cache, request validation built-in, API keys, usage plans, WAF directo.
- Usala para APIs simples Lambda/HTTP cuando NO necesitás las features avanzadas.

### 3. WebSocket API

- Para comunicación **bidireccional en tiempo real** (chat, juegos, dashboards en vivo, trading).
- Mantiene conexión persistente cliente-servidor.
- Backend recibe eventos: `$connect`, `$disconnect`, `$default` y rutas custom.

## Tipos de endpoint (REST API)

OJO que esto es OTRA cosa distinta a "tipo de API". Esto define **DÓNDE se despliega** el endpoint:

| Tipo                         | Dónde vive                                             | Cuándo usarlo                                                               |
| ---------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------- |
| **Edge-Optimized** (default) | Distribuido vía CloudFront en edge locations globales  | Clientes globales — mejora latencia                                         |
| **Regional**                 | En una sola región AWS                                 | Clientes en la misma región — o si vas a poner tu PROPIO CloudFront delante |
| **Private**                  | Accesible solo desde tu VPC vía Interface VPC Endpoint | APIs internas que NO deben salir a internet                                 |

> Trampa clásica: si te preguntan por "API accesible solo desde VPC" → **Private**. Si te preguntan por "API global de baja latencia" → **Edge-Optimized**.

## Integraciones (con qué se conecta el backend)

- **Lambda Function** — la combinación serverless clásica.
- **HTTP endpoint** — cualquier API HTTP pública o interna (ALB, EC2, on-premise).
- **AWS Service** — invocar directo DynamoDB, S3, Step Functions, SQS, Kinesis, sin Lambda en el medio.
- **Mock** — devolver respuestas fijas sin backend (útil para testing).
- **VPC Link** — conectar API Gateway a recursos privados dentro de tu VPC (NLB para REST API, ALB/NLB/Cloud Map para HTTP API).

## Cuándo usar API Gateway

- Frontend público para Lambdas.
- Necesitás auth, throttling, caching delante de un backend.
- APIs REST/HTTP/WebSocket sin gestionar servidores.
- Exponer servicios AWS directamente como API (sin Lambda).

## Cuándo NO usar API Gateway

- Tráfico extremadamente alto y constante donde **ALB + ECS/EC2** puede ser más barato.
- Necesitás **gRPC** → API Gateway NO lo soporta (usá ALB o AppMesh).
- Latencia ultra crítica donde el overhead del Gateway molesta.

## Trampas de examen

1. **REST vs HTTP API** — si te preguntan por la opción **más barata y simple** para Lambda → **HTTP API**. Si necesitás **cache, API keys, usage plans, WAF directo** → **REST API**.
2. **WebSocket** = comunicación bidireccional en tiempo real (NO es para REST).
3. **Endpoint Private** ≠ API privada con auth. Private = accesible SOLO desde VPC.
4. **Edge-Optimized** usa CloudFront por debajo (gestionado por AWS, NO lo ves en tu cuenta).
5. API Gateway **NO soporta gRPC**.

## Auto-test

1. Tenés una Lambda que querés exponer como API REST simple, sin necesidad de cache ni API keys. ¿Qué tipo de API Gateway elegís y por qué?
2. Necesitás una API accesible SOLO desde recursos dentro de tu VPC. ¿Qué tipo de endpoint configurás?
3. Querés construir un chat en tiempo real. ¿Qué tipo de API Gateway usás?
4. Tu cliente está en Europa pero tu backend en us-east-1. ¿Qué tipo de endpoint elegís?
5. ¿Podés invocar DynamoDB directamente desde API Gateway sin pasar por Lambda?

<details>
<summary>Respuestas</summary>

1. **HTTP API** — más barata (~71% menos), más rápida, suficiente para el caso.
2. **Private endpoint** — accesible vía Interface VPC Endpoint, no expuesto a internet.
3. **WebSocket API** — única que soporta conexiones bidireccionales persistentes.
4. **Edge-Optimized** — usa CloudFront para bajar la latencia globalmente.
5. **Sí** — vía integración tipo "AWS Service", directo a DynamoDB sin Lambda en el medio.

</details>
