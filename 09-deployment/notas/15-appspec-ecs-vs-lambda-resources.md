# AppSpec — propiedades requeridas en Resources (ECS vs Lambda)

El archivo `AppSpec` de CodeDeploy **cambia según la plataforma**. Las propiedades de la
sección `Resources` NO son las mismas en ECS que en Lambda. Te meten campos de la otra
plataforma como trampa.

## ECS — Resources requeridos (los TRES)
- **`TaskDefinition`** → qué task definition desplegar.
- **`ContainerName`** → contenedor donde el load balancer enruta tráfico.
- **`ContainerPort`** → puerto de ese contenedor.

(Estos tres van: `TaskDefinition` + `LoadBalancerInfo` { `ContainerName`, `ContainerPort` }.)

### Opcionales en ECS (NO requeridos)
- `PlatformVersion`
- `NetworkConfiguration` (Fargate / awsvpc)
- `CapacityProviderStrategy`

## Lambda — Resources (otra plataforma, otros campos)
- `Name` (nombre de la función)
- `Alias`
- `CurrentVersion`
- `TargetVersion`

## Trampas
- **`alias` / `targetversion`** en una pregunta de ECS → FALSO, son campos del AppSpec de **Lambda**.
- **`NetworkConfiguration`** "requerido" en ECS → FALSO, es **opcional**.
- El `AppSpec` puede ser `.yaml` o `.json`; para EC2/on-prem además lleva hooks de ciclo de vida.

## Ganchos
ECS = TaskDefinition + ContainerName + ContainerPort. Lambda = Name/Alias/CurrentVersion/TargetVersion.
Si ves `alias` o `targetversion` en pregunta de ECS, es trampa de Lambda.

## Pregunta de prueba

Estás escribiendo el `AppSpec` para desplegar un servicio en **ECS** con CodeDeploy.
¿Cuáles propiedades son REQUERIDAS en `Resources`? (Elegí TRES)

A) ContainerPort
B) TaskDefinition
C) alias
D) ContainerName
E) targetversion

<details><summary>Respuesta</summary>

**A, B y D** (ContainerPort, TaskDefinition, ContainerName).
- **C (alias)** y **E (targetversion)** → propiedades del AppSpec de **Lambda**, no de ECS.
- `NetworkConfiguration` y `PlatformVersion` existen en ECS pero son **opcionales**.
</details>
