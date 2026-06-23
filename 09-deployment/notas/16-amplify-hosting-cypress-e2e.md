# Amplify Hosting — CI/CD y pruebas E2E (Cypress)

Para correr pruebas **E2E (Cypress)** en una app frontend con AWS Amplify, el flujo es
CI/CD de **Amplify Hosting**, NO tocar archivos de config del backend.

## Pasos correctos
1. **Conectar el repo de GitHub con Amplify Hosting** → habilita CI/CD; cada push dispara un build.
2. **Actualizar `amplify.yml`** (el buildspec de Amplify) → las pruebas E2E se declaran en la
   fase **`test`** de ese archivo.

## Archivos clave (no confundir)
- **`amplify.yml`** → buildspec del pipeline (fases `backend`, `frontend`, `test`). ACÁ va Cypress.
- **`aws-exports.js`** → AUTOGENERADO. Config del **backend** (endpoints API, Cognito, región)
  que la librería de Amplify lee en el **frontend JS**. No se edita a mano. NADA de testing.
- **`amplifyconfiguration.json`** → equivalente de `aws-exports.js` pero para apps **nativas**
  (iOS/Android/Flutter). También backend config, no Cypress.

## Trampas
- "Incluir config de Cypress en `aws-exports.js`" → FALSO, es archivo generado de backend.
- "Actualizar `amplifyconfiguration.json` para Cypress" → FALSO, es config de backend (nativas).
- "`amplify pull --appId ... --envName ...`" → trae el backend a un entorno **local**; sirve para
  desarrollo, NO para correr E2E en el pipeline.

## Ganchos
E2E en Amplify = conectar GitHub + fase `test` en `amplify.yml`.
`aws-exports.js` / `amplifyconfiguration.json` = backend autogenerado, jamás testing.

## Pregunta de prueba

Una app React en GitHub necesita pruebas E2E con Cypress antes de cada release usando
AWS Amplify. ¿Qué DOS acciones hacés?

A) Conectar el repo de GitHub con AWS Amplify Hosting
B) Incluir la ubicación del archivo de config de Cypress en `aws-exports.js`
C) Actualizar `amplify.yml` con la configuración para Cypress
D) Actualizar `amplifyconfiguration.json` con la configuración para Cypress

<details><summary>Respuesta</summary>

**A y C**.
- **B** falso → `aws-exports.js` es backend autogenerado, no se edita a mano ni guarda config de testing.
- **D** falso → `amplifyconfiguration.json` es el equivalente para apps nativas; tampoco es para Cypress.
- Las E2E van en la fase `test` de `amplify.yml`; conectar GitHub habilita el pipeline.
</details>
