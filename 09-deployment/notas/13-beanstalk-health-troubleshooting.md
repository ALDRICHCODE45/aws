# Elastic Beanstalk — health y troubleshooting

## Enhanced health / Health dashboard

Si un entorno Elastic Beanstalk aparece como `Warning`, `Degraded` o `Severe`,
lo primero que revisás es **Enhanced health / Health dashboard** del entorno.

Ahí podés ver señales agregadas como:

- errores HTTP 4xx/5xx
- health checks fallando
- instancias degradadas
- deployments recientes
- métricas del entorno
- causas probables del estado de salud

## Trampas

- **CloudFormation Change Set** → preview de cambios antes de aplicar; no diagnostica health.
- **CodeBuild `buildspec.yml`** → build/test/package; no diagnostica runtime del entorno.
- **S3 bucket policy** → permisos de S3; solo aplica si el síntoma apunta a S3.
- **SSH manual a cada instancia** → puede servir para deep dive, pero no es el primer paso general.

## Gancho de examen

| Si dice... | Revisar primero |
| ---------- | ---------------- |
| Beanstalk `Severe` / `Degraded` / health checks / 5xx | Enhanced health / Health dashboard |
| Logs de app/servidor sin SSH | `eb logs` / solicitar logs en consola EB |
| Pipeline falla en Build | `buildspec.yml` |
| Deploy con CodeDeploy falla por hooks | `appspec.yml` |

## Pregunta de prueba

Un entorno Elastic Beanstalk aparece como `Severe` tras un deployment. Querés ver
rápidamente si hay instancias degradadas, health checks fallando o aumento de 5xx.
¿Qué revisás primero?

A) Enhanced health / Health dashboard
B) CloudFormation Change Set
C) `buildspec.yml`
D) Bucket policy de S3

<details><summary>Respuesta</summary>

**A**.

Enhanced health / Health dashboard es la vista inicial para diagnosticar estado
del entorno Beanstalk.
</details>
