# Elastic Beanstalk — `.ebextensions` y Blue/Green

## `.ebextensions` = configuración versionada del entorno

Los archivos `.ebextensions/*.config` van dentro del source bundle de Elastic
Beanstalk y se versionan junto al código.

Sirven para configurar:

- opciones del entorno (option settings)
- Auto Scaling
- variables de entorno
- paquetes/comandos
- recursos adicionales de CloudFormation

## Trampa mental

Que una carpeta empiece con punto (`.ebextensions`) NO significa que Git la ignore.

Git solo ignora lo que esté en `.gitignore`.

Ejemplos de carpetas con punto que normalmente sí se versionan:

- `.ebextensions/`
- `.platform/`
- `.github/`

## No confundir archivos

| Archivo / carpeta | Servicio | Para qué |
| ----------------- | -------- | -------- |
| `.ebextensions/*.config` | Elastic Beanstalk | Configurar entorno/recursos |
| `buildspec.yml` | CodeBuild | Comandos de build/test/package |
| `appspec.yml` | CodeDeploy | Archivos y hooks de deployment |

## Blue/Green en Elastic Beanstalk

El patrón típico es:

1. Crear un entorno nuevo con la nueva versión.
2. Validarlo.
3. Intercambiar los CNAMEs entre el entorno viejo y el nuevo.

El CNAME swap mueve el tráfico rápidamente sin actualizar el entorno viejo.

## Trampas

- Cambiar la policy del entorno viejo a Rolling NO hace blue/green.
- Terminar el entorno viejo antes de mover tráfico corta la app.
- `sam deploy` es para SAM/serverless, no para Beanstalk.

## Error propio: Blue/Green ≠ Rolling policy

Si el enunciado dice que ya creaste un **entorno nuevo** y querés mover el
tráfico al nuevo después de validarlo, la acción es **CNAME swap**.

No cambies la deployment policy del entorno viejo a Rolling:

- **Rolling** actualiza un entorno existente por batches.
- **Blue/Green** mueve tráfico entre dos entornos separados.

Gancho de examen:

| Si dice... | Respuesta |
| ---------- | --------- |
| "nuevo entorno validado" + "mover tráfico" | **CNAME swap** |
| "actualizar el mismo entorno por batches" | **Rolling / Rolling + batch** |

## Pregunta de prueba

Querés configurar opciones de entorno y recursos adicionales en Elastic Beanstalk
de forma versionada con el código. Además, para un blue/green, querés mover el
tráfico al entorno nuevo después de validarlo. ¿Qué conceptos usás?

A) `appspec.yml` + terminar el entorno viejo
B) `.ebextensions/*.config` + CNAME swap
C) `buildspec.yml` + `sam deploy`
D) EventBridge + Rolling deployment

<details><summary>Respuesta</summary>

**B**.

- `.ebextensions/*.config` configura el entorno Beanstalk desde el source bundle.
- CNAME swap mueve tráfico entre entornos en un blue/green.

Cuándo sería cada una:

- `appspec.yml` → CodeDeploy, no Beanstalk.
- `buildspec.yml` → CodeBuild, no configuración EB.
- `sam deploy` → SAM/serverless, no Beanstalk.
- Rolling → actualiza un entorno existente; no es blue/green entre dos entornos.
</details>
