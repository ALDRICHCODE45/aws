# KMS — Envelope Encryption APIs

| API | Devuelve | Cuándo |
| --- | -------- | ------ |
| `GenerateDataKey` | Plaintext + Encrypted | Cifrar **AHORA** |
| `GenerateDataKeyWithoutPlaintext` | Solo Encrypted | Cifrar **DESPUÉS** (pedís Decrypt cuando lo necesites) |
| `Encrypt` | Cifra datos pequeños (**<4 KB**) directo | Secreto chico, NO archivos |
| `Decrypt` | Plaintext de una key cifrada | Recuperar plaintext para usarlo |
| `GenerateRandom` | Bytes aleatorios | Randomness, NO cifrado |

## Disparadores

- "**ahora**" / "cifrar inmediatamente" → `GenerateDataKey`
- "**en un momento posterior**" / "después" / "guardar solo cifrada" → `GenerateDataKeyWithoutPlaintext`
- "datos pequeños" / "<4 KB" → `Encrypt`

## Trampa

`WithoutPlaintext` parece "no me sirve" pero ES la opción **más segura cuando no cifrás YA**. Plaintext vive en memoria solo cuando lo necesitás (vía Decrypt).
