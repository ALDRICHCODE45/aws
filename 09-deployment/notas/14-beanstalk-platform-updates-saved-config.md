# Elastic Beanstalk — Managed platform updates y Saved configurations

## Managed platform updates

Elastic Beanstalk puede aplicar automáticamente actualizaciones menores/patches de
plataforma durante una ventana de mantenimiento configurada.

Usalo cuando el enunciado dice:

- reducir carga operativa
- aplicar patches automáticamente
- ventana de mantenimiento
- actualizaciones menores de plataforma

## Trampas

- **Traffic splitting** → deployment policy/canary de una versión de la app, no patching de plataforma.
- **`.ebextensions` con `yum update`** → manual/frágil; no es la feature administrada.
- **CodeDeploy Canary** → CodeDeploy/Lambda/ECS, no Beanstalk platform updates.

## Saved configuration

Una **Saved configuration** guarda la configuración de un entorno Beanstalk para
reutilizarla después al crear otro entorno similar.

Puede incluir opciones como:

- environment properties / variables
- capacity
- load balancer
- scaling
- health settings
- otras option settings del entorno

Usalo cuando el enunciado dice:

- guardar configuración actual del entorno
- reutilizar config al crear otro entorno
- crear entornos similares con la misma configuración

## No confundir

| Necesidad | Respuesta |
| --------- | --------- |
| Patches automáticos de plataforma en ventana | Managed platform updates |
| Reutilizar configuración de un entorno | Saved configuration |
| Preview de cambios CloudFormation | Change Set |
| Hooks/archivos CodeDeploy | `appspec.yml` |
| Permiso temporal KMS | KMS grant |

## Pregunta de prueba

Querés que Beanstalk aplique patches menores de plataforma automáticamente en una
ventana de mantenimiento, y también guardar la configuración actual del entorno
para crear otro igual más adelante. ¿Qué usás?

A) Managed platform updates + Saved configuration
B) Traffic splitting + Change Set
C) `.ebextensions` con `yum update` + `appspec.yml`
D) CodeDeploy Canary + KMS grant

<details><summary>Respuesta</summary>

**A**.

- Managed platform updates aplica patches menores automáticamente.
- Saved configuration guarda la config del entorno para reutilizarla.
</details>
