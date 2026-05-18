# Lambda — Objeto de evento y objeto de contexto

Resumen rápido para examen y práctica.

---

## Objeto de evento (Event Object)

Es el **payload** que recibe la función Lambda.

- Contiene los datos que la función debe procesar.
- Su forma cambia según el origen del evento.
- Puede incluir metadatos del origen.

Ejemplos de origen:

- **Amazon Simple Storage Service (Amazon S3)**
- **Amazon Simple Queue Service (Amazon SQS)**
- **Amazon API Gateway**

---

## Objeto de contexto (Context Object)

Contiene metadatos de la invocación actual y del entorno de ejecución.

Ejemplos comunes:

- Identificador de solicitud (`awsRequestId`)
- Nombre y versión de la función
- Límite de memoria configurado
- Tiempo restante antes del timeout (`getRemainingTimeInMillis` en Node.js)

---

## Regla mental

- **Event Object** = qué trabajo hacer
- **Context Object** = en qué condiciones se ejecuta ese trabajo

---

## Nota para Go

En Go, el primer parámetro `context.Context` es el contexto estándar del lenguaje.
Para datos específicos de Lambda (por ejemplo `awsRequestId`) se usa `lambdacontext` del SDK de Lambda.
