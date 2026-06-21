# S3 — Developer Essentials (DVA-C02)

## Presigned URLs

- URL temporal y FIRMADA a un objeto privado, sin hacer público el bucket.
- Sirve para **download (GET) Y upload (PUT)** ← trampa: no es solo download.
- Quien la recibe hereda los **permisos del que la firmó**.
- Tiene **expiración** (SDK default ~1h, configurable).
- Disparadores: "subir/bajar sin dar credenciales AWS y sin bucket público" → **Presigned URL**.
- Distractores: hacer bucket público / mandar access keys al cliente → MAL.

## S3 Event Notifications

- Eventos tipo `s3:ObjectCreated:*`, `s3:ObjectRemoved:*`.
- Destinos clásicos: **SNS, SQS, Lambda**.
- Moderno: **EventBridge** → filtros avanzados, múltiples destinos, replay.
- "procesar imagen al subir" → S3 Event → Lambda.
- "filtros avanzados / muchos destinos" → EventBridge.

## Cifrado (4 tipos)

| Tipo         | Key la maneja                            | Cuándo                                               |
| ------------ | ---------------------------------------- | ---------------------------------------------------- |
| **SSE-S3**   | AWS (AES-256)                            | default, simple                                      |
| **SSE-KMS**  | KMS / tu CMK                             | auditoría (CloudTrail) + rotación + permisos por key |
| **SSE-C**    | VOS (en cada request, HTTPS obligatorio) | key fuera de AWS                                     |
| **DSSE-KMS** | KMS doble capa                           | compliance extremo (raro)                            |

### Trampas cifrado

- "auditar quién desencripta / rotación / control fino" → **SSE-KMS**.
- SSE-KMS llama a KMS en cada GET/PUT → con alto tráfico = **throttling de KMS**.
  Solución: **S3 Bucket Keys** (reduce llamadas a KMS). ESTO CAE.
- "cliente manda su propia clave en cada request" → **SSE-C**.

## Multipart upload

- **> 5 GB**: multipart OBLIGATORIO. **> 100 MB**: recomendado.
- Subidas incompletas dejan partes basura → **Lifecycle rule para abortar
  multipart incompletos** (limpia + ahorra costo). Cae.

## Otros que caen

- Versioning: protege de borrados/sobrescrituras. MFA Delete = capa extra.
- Read-after-write: hoy S3 es **strongly consistent** (no más "eventual" en lecturas).

## Pregunta de prueba

Una app móvil necesita que los usuarios suban fotos directo a un bucket S3
privado, de forma segura, sin exponer credenciales AWS. ¿Qué usás?

A) Hacer el bucket público con una bucket policy
B) Generar una Presigned URL (PUT) y dársela al cliente
C) Enviar las access keys de un rol IAM al cliente
D) Activar SSE-C en el bucket

<details><summary>Respuesta</summary>

**B**: Presigned URL para PUT (sirve para upload y download, expira, sin bucket público).
Cuándo sería cada una:
- **bucket público / access keys al cliente** → nunca, son malas prácticas.
- **SSE-C** → es un tipo de cifrado (el cliente trae la clave), no resuelve el upload.
</details>
