# S3 — Cifrado obligatorio vía Bucket Policy

## Concepto (qué es el header)

- **SSE** = S3 cifra el objeto en reposo por vos (server-side = lo hace AWS).
- El **header** `x-amz-server-side-encryption` es la INSTRUCCIÓN de qué cifrado usar
  en el `PutObject`. NO es la clave, es la "etiqueta" del sobre.
- `AES256` → SSE-S3 (AWS maneja todo). `aws:kms` → SSE-KMS (auditoría + control,
  usa envelope encryption). `customer` en el header → SSE-C (vos traés la clave).
- La bucket policy FUERZA cifrado **rechazando (Deny)** los PutObject sin el header.
  Se usa `Deny` (no `Allow`) porque un Deny explícito SIEMPRE gana.

## Tipos SSE → header

| Tipo                          | Header                                                                  |
| ----------------------------- | ----------------------------------------------------------------------- |
| SSE-S3                        | `x-amz-server-side-encryption: AES256`                                  |
| SSE-KMS                       | `x-amz-server-side-encryption: aws:kms`                                 |
| SSE-KMS con CMK específica    | `x-amz-server-side-encryption-aws-kms-key-id`                           |
| **SSE-C** (clave del cliente) | `x-amz-server-side-encryption-customer-key` + `-algorithm` + `-key-MD5` |

> ⚠️ **Si ves `customer` en el header → es SSE-C**, NO SSE-S3 ni KMS. Si la pregunta pide SSE-S3 o SSE-KMS y la opción tiene `customer` → descarte automático.

## Trampas claves

1. **`s3:PostObject` NO EXISTE.** La única acción para subir es `s3:PutObject`. Si la ves → descarte.
2. **AES256 = SSE-S3**, NO KMS. Si la pregunta pide KMS y la opción tiene AES256 → mal.
3. **Más específico gana**: si pide SSE-KMS, usar el header `-aws-kms-key-id` (no el genérico).

## Lógica de policy — "forzar" cifrado

```
Para FORZAR algo:    Deny + AUSENCIA del header  (Null: true / StringNotEquals)
Para PROHIBIR algo:  Deny + PRESENCIA del header (Null: false / StringEquals)
```

> Bucket policies para "obligar cifrado" siempre son `Deny + condición de AUSENCIA`. Si ves "Deny + condición de PRESENCIA" → al revés de lo que querés.

## Pregunta de prueba

Una empresa exige que TODOS los objetos subidos a un bucket usen cifrado con KMS
para tener auditoría en CloudTrail. ¿Qué configurás?

A) Bucket policy con `Allow s3:PutObject` si el header es `aws:kms`.
B) Bucket policy con `Deny s3:PutObject` cuando el header NO sea `aws:kms`.
C) Bucket policy con `Deny s3:PutObject` cuando el header sea `AES256`.
D) Activar SSE-C en el bucket.

<details><summary>Respuesta</summary>

**B**. Forzar cifrado = `Deny` + AUSENCIA del valor deseado (`StringNotEquals: aws:kms`).

- **A** mal: con `Allow` alguien con otro permiso se lo saltea; no garantiza nada.
- **C** mal: solo bloquea AES256, pero deja pasar objetos SIN cifrado.
- **D** mal: SSE-C es la clave del cliente, no da auditoría en CloudTrail; piden KMS.

Cuándo sería cada una:

- "auditar quién descifra / control de claves" → **SSE-KMS** (`aws:kms`).
- "lo más simple, sin gestionar nada" → **SSE-S3** (`AES256`).
- "el cliente trae su propia clave" → **SSE-C** (header `customer`).
</details>
