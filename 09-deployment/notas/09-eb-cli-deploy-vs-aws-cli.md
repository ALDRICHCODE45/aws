# Elastic Beanstalk — deploy por CLI (eb deploy)

## Respuesta correcta
Empaquetar como **ZIP** + desplegar con **`eb deploy`** (EB CLI).

## Dos ejes
1. **Formato**: **ZIP** (o **WAR** para Java). NUNCA TAR → descartar opciones TAR.
2. **Comando**: **`eb deploy`** (EB CLI), no AWS CLI.

## EB CLI vs AWS CLI
| CLI | Deploy |
| --- | ------ |
| **EB CLI** (`eb`) | **`eb deploy`** → zipea + sube a S3 + crea app version + despliega. TODO. |
| **AWS CLI** (`aws elasticbeanstalk`) | `create-application-version` + `update-environment --version-label` (2 pasos) |

## Trampa de nombre (familia recurrente)
`aws elasticbeanstalk update-application` **NO despliega**. Solo cambia metadatos
de la aplicación (descripción, etc.). Suena a "actualizar mi app = deploy" pero NO.
Misma familia que: s3:PostObject, "programar desde consola Lambda",
"VisibilityTimeout descarta duplicados".

## Regla anti-nombre-engañoso
Si un comando/feature SUENA perfecto → preguntá "¿hace EXACTAMENTE eso o solo
suena bien?". El examen vive de nombres que prometen lo que no cumplen.

## Pregunta de prueba

¿Cuál es la forma correcta de desplegar una nueva versión en Elastic Beanstalk
usando la CLI?

A) Empaquetar como TAR y usar `eb deploy`
B) Empaquetar como ZIP y usar `eb deploy`
C) Empaquetar como ZIP y usar `aws elasticbeanstalk update-application`
D) Empaquetar como TAR y usar `aws elasticbeanstalk update-application`

<details><summary>Respuesta</summary>

**B**: ZIP (o WAR para Java) + `eb deploy` (la EB CLI hace todo el flujo).
Cuándo sería cada una:
- **update-application** → solo cambia metadatos de la app, NO despliega (trampa de nombre).
- **TAR** → nunca: Beanstalk usa ZIP/WAR.
- Con AWS CLI deployás con `create-application-version` + `update-environment`, no con update-application.
</details>
