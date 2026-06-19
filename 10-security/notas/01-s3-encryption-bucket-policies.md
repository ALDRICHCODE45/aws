# S3 — Cifrado obligatorio vía Bucket Policy

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
