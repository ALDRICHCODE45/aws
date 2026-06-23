# SSE-KMS + multipart upload → requiere kms:Decrypt (gotcha)

Gotcha contraintuitivo: subir un archivo GRANDE a un bucket S3 con **SSE-KMS** puede
fallar con **access denied**, mientras que el mismo `aws s3 cp` con un archivo chico funciona.

## Por qué (la pista es el TAMAÑO)
1. El **AWS CLI sube archivos grandes con multipart upload** automáticamente (los parte en trozos).
2. Con SSE-KMS, cada parte se cifra con la data key; para **ensamblar/completar** el objeto
   final S3 debe **descifrar** → requiere **`kms:Decrypt`**.
3. Por eso, para SUBIR a un bucket SSE-KMS necesitás:
   **`kms:GenerateDataKey` + `kms:Encrypt` + `kms:Decrypt`** (el Decrypt es el sorprendente).
4. Archivo chico = single PUT, no ejerce ese camino → pasa sin error.
   Archivo grande = multipart → sin `kms:Decrypt` = access denied.

(Conecta con envelope encryption: una data key debe estar en texto plano para operar;
reusarla en el ensamblaje exige pasarla por `Decrypt`.)

## Dónde van los permisos KMS
- En la **política IAM del usuario / key policy**, NO en la **bucket policy**.

## Trampas
- "Falta `kms:Encrypt` en la **bucket policy**" → MAL: permisos KMS no van en bucket policy;
  y si faltara Encrypt fallarían TODOS los tamaños, no solo el grande.
- "Bucket policy que bloquea cargas mayores a 75GB" → inventado.
- "KMS solo cifra hasta 50GB" → límite falso.

## Ganchos
S3 chico OK + grande falla con SSE-KMS = multipart → falta **`kms:Decrypt`** en la IAM policy.
Subir a SSE-KMS = GenerateDataKey + Encrypt + Decrypt.

## Pregunta de prueba

`aws s3 cp` de un archivo de 75 GB a un bucket SSE-KMS da **access denied**, pero con
archivos de 5 GB funciona. ¿Cuáles son DOS causas posibles?

A) Falta `kms:Encrypt` en la política del **bucket**
B) Una bucket policy bloquea cargas mayores a 75 GB
C) KMS solo cifra archivos de hasta 50 GB
D) El CLI hace multipart para archivos grandes, que requiere permisos KMS adicionales
E) Falta `kms:Decrypt` en la política **IAM** del desarrollador

<details><summary>Respuesta</summary>

**D y E**. Multipart (archivos grandes) necesita `kms:Decrypt` para ensamblar; el chico va single PUT.
- **A** → permisos KMS no van en bucket policy; y afectaría TODOS los tamaños.
- **B** → inventado (75GB ni siquiera es "mayor a 75GB").
- **C** → límite falso.
</details>
