# Amazon Macie — descubrir datos sensibles en S3

**Macie** = servicio de seguridad de datos que usa **machine learning** para descubrir,
clasificar y proteger **datos sensibles en Amazon S3** (PII, financieros, credenciales).
Automático y escalable (sirve para decenas de buckets con terabytes).

## Para qué se usa
- Identificar **dónde** hay datos sensibles expuestos en S3.
- Genera **findings** con tipos gestionados (managed finding types):
  - `SensitiveData:S3Object/Financial` → datos **financieros**
  - `SensitiveData:S3Object/Personal` → datos **personales (PII)**
  - `SensitiveData:S3Object/Credentials` → credenciales/secretos
  - `SensitiveData:S3Object/CustomIdentifier`, `.../Multiple`

## Trampas
- Pregunta sobre datos **financieros** pero la opción usa el tipo **`/personal`** → MAL,
  hay que usar **`/financial`**. Mismo servicio, finding type equivocado.
- **Inspección manual** de cada bucket → no escala con terabytes.
- **CloudTrail** → rastrea **actividad de API** (quién hizo qué), NO clasifica el **contenido**
  de los datos. No sirve para encontrar datos sensibles dentro de objetos.

## Ganchos
"Encontrar/clasificar datos sensibles en S3, automático y escalable" → **Macie**.
Macie clasifica CONTENIDO; CloudTrail rastrea ACTIVIDAD. No confundir.

## Pregunta de prueba

Una org tiene decenas de buckets S3 con terabytes y necesita identificar
automáticamente dónde se exponen **datos financieros** sensibles. ¿Qué hacés?

A) Macie con finding type `SensitiveData:S3Object/Personal` en todos los buckets
B) Macie con finding type `SensitiveData:S3Object/Financial` en todos los buckets
C) Inspeccionar manualmente cada bucket
D) Usar logs de CloudTrail para rastrear transacciones de datos financieros

<details><summary>Respuesta</summary>

**B**: Macie con el finding type **`/Financial`** (piden datos financieros).
- **A** → `/Personal` detecta PII, no financieros (trampa de finding type).
- **C** → no escala con terabytes.
- **D** → CloudTrail rastrea actividad de API, no clasifica el contenido de los objetos.
</details>
