# Lambda — Directorio /tmp

Resumen corto para examen.

---

## Qué es

Directorio temporal en disco que tiene cada contenedor de **AWS Lambda**.
Sirve para guardar archivos durante la ejecución.

---

## Características

- Tamaño por defecto: **512 MB**.
- Configurable hasta **10 GB** (`ephemeral storage`).
- Es **local al contenedor**: no se comparte entre contenedores.
- Es **efímero**: cuando el contenedor muere, `/tmp` muere con él.

---

## Caché transitoria entre invocaciones

El contenido de `/tmp` **persiste mientras el contenedor está congelado** (frozen) entre invocaciones.
Mismo principio que las variables globales, pero en disco.

Flujo típico:

1. Invocación #1 (cold start): Lambda descarga un archivo grande de **Amazon Simple Storage Service (Amazon S3)** a `/tmp/datos.json`.
2. Lambda responde. Contenedor se congela.
3. Invocación #2 (warm start, mismo contenedor): el archivo SIGUE en `/tmp`.
4. El código chequea si existe y evita bajarlo de nuevo.

---

## Cuidado

- No es confiable como almacenamiento persistente.
- Si la invocación cae en OTRO contenedor (Lambda escala horizontal), `/tmp` está vacío.
- Si AWS mata el contenedor, `/tmp` se pierde.
- Siempre chequear `existe el archivo?` antes de usarlo.

---

## Cuándo usar /tmp

- Descomprimir archivos grandes.
- Procesar imágenes, videos, PDFs.
- Cache local de archivos pesados que se reusan entre invocaciones del mismo contenedor.

---

## Cuándo NO usar /tmp

- Compartir datos entre contenedores distintos → usar **Amazon S3**, **Amazon DynamoDB** o **Amazon Elastic File System (Amazon EFS)** montado en Lambda.
- Persistencia garantizada → usar S3 o base de datos.

---

## Punto de examen

- Tamaño: **512 MB default, hasta 10 GB**.
- `/tmp` persiste entre invocaciones del MISMO contenedor (caché transitoria).
- NO se comparte entre contenedores.
- Para storage compartido entre Lambdas: **EFS**, **S3** o **DynamoDB**.
