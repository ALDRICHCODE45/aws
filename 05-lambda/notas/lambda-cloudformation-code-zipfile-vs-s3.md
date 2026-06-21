# Lambda en CloudFormation — Code.ZipFile vs S3

## Estructura (AWS::Lambda::Function)

```yaml
Properties:
  Code:
    ZipFile: | # código INLINE (sí, se llama ZipFile)
      exports.handler = async () => 'Hello World'
```

o para S3:

```yaml
Code:
  S3Bucket: mi-bucket
  S3Key: funcion.zip
```

## Reglas

- **Código inline** → `Code.ZipFile`. La propiedad es `Code`, el código va en `ZipFile`.
  NO se vuelca el código directo en `Code` (es un objeto).
- **Código desde S3** → `Code.S3Bucket` + `Code.S3Key`. (ZipFile NO es una ruta S3.)

## Límites del inline ZipFile

- Solo runtimes interpretados: **Node.js y Python** (NO Java/Go/compilados).
- Máximo **4096 caracteres (4 KB)** → más grande = obligatorio S3.
- Solo AWS SDK incluido; sin dependencias externas.

## Pregunta tipo

"forma más SENCILLA de desplegar un Hello World Node.js con CloudFormation"
→ inline **`Code.ZipFile`** (sin subir nada a S3).

## Trampas

- "ZIP a S3 + ruta en ZipFile" → MAL: ZipFile es inline, no ruta S3.
- "código directo en parámetro Code" → MAL: va en Code.ZipFile, no en Code.
- "S3Key + S3Bucket" → funciona pero más pasos; no es "lo más sencillo" para algo trivial.

## Pregunta de prueba

¿Cuál es la forma MÁS SENCILLA de desplegar una Lambda Node.js que devuelve
'Hello World' usando CloudFormation?

A) Incluir el código directamente en el parámetro `Code`
B) Incluir el código en `Code.ZipFile` (inline)
C) Subir un ZIP a S3 y poner la ruta en `Code.ZipFile`
D) Subir a S3 y usar `Code.S3Bucket` + `Code.S3Key`

<details><summary>Respuesta</summary>

**B** (`Code.ZipFile` inline): para algo trivial, sin subir nada a S3 (Node.js/Python, ≤4KB).
Cuándo sería cada una:

- **Code directo** → no existe; el código va en `Code.ZipFile`, no en `Code`.
- **ZIP a S3 + ZipFile** → mal: ZipFile es inline, no una ruta S3.
- **S3Bucket + S3Key** → cuando el código es grande (>4KB) o compilado (Java/Go).
</details>
