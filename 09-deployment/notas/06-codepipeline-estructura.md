# CodePipeline — estructura y trampas

## Qué es

Orquestador CI/CD. Se divide en **stages** (Source → Build → Deploy...) y cada
stage tiene **actions**. Los artefactos viajan entre stages por un **bucket S3**
(artifact store), normalmente cifrado con **KMS**.

## Cadena típica

Source (CodeCommit / GitHub / S3) → Build (CodeBuild) → Deploy
(CodeDeploy / CloudFormation / ECS / Beanstalk).

## Disparo del pipeline

- Buenas prácticas: **EventBridge (CloudWatch Events) rule** detecta el commit y dispara.
- GitHub: webhook.
- Evitar polling periódico (más lento, peor).

## Trampas de examen

- "El pipeline NO arranca solo al hacer commit" → falta la **EventBridge rule** / webhook.
- "Falla al pasar artefacto entre stages" → permisos del **rol** sobre el
  **bucket S3** del artifact store o el **KMS** que lo cifra.
- "Falla solo el stage de Build" → mirar **CodeBuild** (buildspec / permisos / imagen).

## Archivos clave (NO confundir)

- **buildspec.yml** → CodeBuild. Define fases: install / pre_build / build / post_build.
  Va en la **raíz** del repo (o se especifica la ruta).
- **appspec.yml** → CodeDeploy. Define qué/cómo desplegar + hooks.
- Trampa: buildspec = construir; appspec = desplegar. No los mezcles.

## Pregunta de prueba

Tu CodePipeline NO arranca solo cuando hay un commit en CodeCommit. ¿Qué falta?

A) Un archivo buildspec.yml
B) Una regla de EventBridge (CloudWatch Events) que detecte el commit
C) Un archivo appspec.yml
D) Más shards en el artifact store

<details><summary>Respuesta</summary>

**B**. El disparo automático lo hace una **EventBridge rule** (o webhook en GitHub).
Cuándo sería cada una:
- **buildspec.yml** → si fallara/faltara el stage de **Build** (CodeBuild).
- **appspec.yml** → si fallara el **Deploy** (CodeDeploy).
- "Falla al pasar artefacto entre stages" → permisos del rol sobre el **bucket S3/KMS**.
</details>
