# Elastic Beanstalk — logs, environment properties y worker environments

## Logs en Elastic Beanstalk

Para revisar logs de una app Beanstalk sin entrar por SSH a cada instancia:

- Consola de Elastic Beanstalk → solicitar logs del entorno.
- EB CLI → `eb logs`.

Esto sirve para recuperar logs de aplicación/servidor desde las instancias del
entorno.

## Trampas

- **Change Set** → CloudFormation preview; no recupera logs.
- **X-Ray** → tracing distribuido; no reemplaza pedir logs del entorno.
- **`sam local invoke`** → SAM/Lambda local; no es Elastic Beanstalk.

## Environment properties / variables de entorno

Para configurar valores que cambian por ambiente (`dev`, `staging`, `prod`) sin
hardcodearlos en el código, usá **Environment properties** del entorno Beanstalk.

Ejemplo típico:

- `API_ENDPOINT`
- `DB_HOST`
- `FEATURE_FLAG_X`

La aplicación los lee como variables de entorno.

## Trampa: configuración por ambiente ≠ archivo de deploy

Si el enunciado dice que un valor cambia por ambiente y la app debe leerlo sin
hardcodear, la respuesta natural es **Environment properties**, no `appspec.yml`,
AMI hardcodeada ni Change Set.

| Necesidad | Respuesta |
| --------- | --------- |
| Valor distinto por `dev/staging/prod` | Environment properties |
| Configurar recursos/opciones EB versionadas | `.ebextensions/*.config` |
| Hooks de despliegue en EB Amazon Linux 2 | `.platform/hooks/` |
| Hooks de CodeDeploy | `appspec.yml` |

## `.platform/` en Amazon Linux 2

En plataformas modernas de Elastic Beanstalk basadas en Amazon Linux 2, usá
`.platform/` para configuración específica de plataforma dentro del source
bundle.

Casos típicos:

- `.platform/hooks/` → scripts durante fases del deployment.
- `.platform/nginx/` → configuración de NGINX/proxy.

No confundir:

- `.ebextensions/*.config` → option settings, paquetes, comandos y recursos
  CloudFormation extra.
- `.platform/` → configuración moderna de plataforma/proxy/hooks en AL2.
- `appspec.yml` → CodeDeploy, no Beanstalk.

Gancho de examen:

| Si dice... | Respuesta |
| ---------- | --------- |
| Beanstalk + Amazon Linux 2 + hooks de deploy | `.platform/hooks/` |
| Beanstalk + Amazon Linux 2 + NGINX/proxy | `.platform/` |
| Beanstalk + recursos/opciones del entorno versionadas | `.ebextensions/*.config` |

## Worker environment

Para trabajos en background/asíncronos en Beanstalk, usá un **Worker

- Web server environment → tráfico HTTP normal detrás de ALB/ELB.
- Worker environment → consume mensajes de SQS para procesar jobs.
- Immutable / traffic splitting / rolling → son políticas de deployment, no tipos
  de entorno.

## Pregunta de prueba

Una app Beanstalk falla en producción y querés revisar logs sin SSH. Además,
necesita un `API_ENDPOINT` distinto por ambiente. ¿Qué usás?

A) `eb logs` + Environment properties
B) Change Set + `appspec.yml`
C) `sam local invoke` + AMI hardcodeada
D) X-Ray obligatorio + Traffic splitting

<details><summary>Respuesta</summary>

**A**.

- `eb logs` / consola EB permite solicitar logs del entorno.
- Environment properties configura variables por entorno sin hardcodear.

Cuándo serían las otras:

- Change Set → preview de CloudFormation.
- `appspec.yml` → CodeDeploy.
- `sam local invoke` → SAM/Lambda local.
- X-Ray → tracing; útil para observabilidad, pero no es la forma básica de pedir logs EB.
</details>
