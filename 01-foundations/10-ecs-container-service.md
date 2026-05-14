# ECS — Elastic Container Service

Apunte de consolidación con foco de examen (DVA-C02).
Basado en la práctica del proyecto backend + conceptos de certificación.

---

## 1) Mapa mental en 20 segundos

- **Cluster** = grupo de recursos (EC2 o Fargate) donde corren tus servicios.
- **Service** = mantiene N réplicas de una task corriendo (auto-healing).
- **Task Definition** = plantilla/blueprint: imagen, CPU, RAM, red, logs, IAM role.
- **Task** = contenedor en ejecución (la instancia real de tu app).

```
Cluster
 └── Service (mantiene N tasks vivas)
      └── Task Definition (la receta)
           ├─ Container definitions (imagen, puertos, env vars, logs)
           ├─ CPU / Memory (en la task, no en el container)
           ├─ Network mode (awsvpc recomendado)
           └─ IAM Task Role
```

**Si te preguntan "¿qué lanza los contenedores?" → Task Definition es la respuesta.**

---

## 2) Launch Types: EC2 vs Fargate

| | **EC2** | **Fargate** |
|---|---|---|
| Infra | Vos provisionás las EC2 en el cluster | AWS maneja los servidores por vos |
| Escalabilidad | Escalás instancias EC2 + tasks | Solo definís tasks, AWS escala infra |
| Mantenimiento | Tú: SO, parches, actualizaciones | AWS lo hace |
| Acceso SSH | Sí (a las EC2) | No existe |
| Control | Granular (instancias grandes, GPU, placement groups) | Solo CPU/RAM por task |
| Caso típico examen | Cargas pesadas, legacy, GPUs | La mayoría de escenarios modernos |

**Truco examen**: Si la pregunta dice "sin gestionar servidores" / "sin parchar SO" / "quiero solo desplegar contenedores" → **Fargate**.

---

## 3) Networking

### awsvpc (el modo que usaste y el más importante)
- Cada task obtiene su **propia ENI** con IP privada.
- Se puede asociar un **Security Group** específico a cada task.
- **Requerido para ALB/NLB** (el modo bridge no funciona bien con load balancers).

### Fargate + awsvpc: el detalle clave (Load Balancing)
En Fargate **no existe el concepto de "puerto del host"** porque no hay host que vos gestionés.
En Docker clásico hacés `-p 8080:80` (host:contenedor). En Fargate eso no aplica:

```
Docker clásico:  host:8080 → contenedor:80   (mapeo necesario)
Fargate awsvpc:  task-ip:80 → contenedor:80  (directo, sin mapeo)
```

- El ALB habla directo con la IP privada de cada task en el puerto del contenedor.
- Por eso en la Task Definition **solo definís el puerto del contenedor**, no el del host.
- Cada task tiene su propia IP → el ALB puede enrutar a cada una de forma independiente.

**Patrón de Security Groups en Load Balancing Fargate:**
```
SG-ALB   → inbound 80/443 desde 0.0.0.0/0 (internet)
SG-Tasks → inbound 80 solo desde SG-ALB (no desde internet directamente)
```
Así solo el ALB puede hablar con las tasks → las tasks nunca quedan expuestas directamente.

### Otros modos (menos relevantes para examen)
- **bridge**: el default legacy, comparte la ENI del host.
- **host**: usa la red de la EC2 directamente (raro).
- **none**: sin red (solo para tareas batch sin red).

### Trampa clásica: puerto host fijo en EC2 launch type (bridge)

En EC2 launch type con network mode `bridge`, cada contenedor necesita mapear su puerto interno al puerto del host EC2:

```
Contenedor 1: host:8080 → contenedor:80  ✅
Contenedor 2: host:8080 → contenedor:80  💥 CONFLICTO — puerto ya ocupado
```

Si intentás correr **dos copias del mismo contenedor en la misma EC2** con un puerto host fijo, el segundo falla — aunque sobre CPU y RAM.

**Solución: puerto host = 0 (o vacío) en la Task Definition**

AWS asigna un puerto aleatorio disponible (ephemeral port) a cada contenedor:
```
Contenedor 1: host:32771 → contenedor:80  ✅
Contenedor 2: host:32772 → contenedor:80  ✅
```
El ALB descubre esos puertos dinámicos automáticamente → **Dynamic Port Mapping**.

**Por qué Fargate no tiene este problema:**
En Fargate (`awsvpc`) cada task tiene su **propia IP privada** — no hay host compartido, no existe el concepto de puerto host. Por eso solo definís el puerto del contenedor y listo.

| | **EC2 + bridge** | **Fargate + awsvpc** |
|---|---|---|
| Puerto host | Necesario (puede colisionar) | No existe |
| Múltiples tasks en mismo host | Requiere puerto host = 0 | Sin problema (IPs separadas) |
| Solución conflicto | Dynamic Port Mapping | N/A |

---

## 4) Security Groups (refresco)

En tu práctica quedó claro:

```
SG-ALB       → inbound 443 desde 0.0.0.0/0
SG-Tasks     → inbound [puerto-app] solo desde SG-ALB
```

- **Stateful**: si entra tráfico permitido, la respuesta sale automáticamente.
- Se puede referenciar **un SG dentro de otro** → desacoplamiento de IPs.
- Múltiples SG en una ENI → AWS evalúa la **unión** (OR) de reglas.

---

## 5) Service Auto Scaling

| Tipo | Cuándo usar |
|---|---|
| **Target Tracking** | Mantener CPU/memoria en un valor (ej: 50%). El más común. |
| **Step Scaling** | Escalar por pasos según alarmas CloudWatch (ej: +2 tasks si CPU > 70%). |
| **Scheduled** | Horarios predecibles (ej: más tasks en horario pico). |

- Escala la **cantidad de tasks** dentro de un Service.
- **Mínimo** y **máximo** de tasks son obligatorios al configurar.
- **No confundir** con EC2 Auto Scaling (que escala las instancias del cluster).

**Truco examen**: Si la pregunta dice "scale the number of running tasks based on CPU" → **ECS Service Auto Scaling con Target Tracking**.

---

## 6) Estrategias de Despliegue

### Rolling Update (default)
1. Se detienen tasks antiguas **de a una** (o en lotes configurables).
2. Se lanzan nuevas tasks con la definición actualizada.
3. Se espera que pase el health check antes de detener las siguientes.
4. **Puede haber breve degradación** durante el rollout.

### Blue/Green (con AWS CodeDeploy)
- Se crea un **nuevo servicio** con la versión nueva.
- Se valida (pruebas, canary traffic).
- Se cambia el DNS/ALB target group al nuevo servicio.
- **Zero downtime**.

### Canary
- Un porcentaje del tráfico va a la nueva versión.
- Si funciona, se migra todo.

### Regla examen
| Pregunta dice... | Respuesta |
|---|---|
| "minimize downtime" | Blue/Green |
| "cost-effective, simple" | Rolling Update |
| "small percentage of traffic first" | Canary |

---

## 7) ECR — Elastic Container Registry

- Es el **Docker registry** gestionado por AWS.
- Almacena, versionea y escanea imágenes Docker.
- Las task definitions referencian imágenes de ECR (ej: `123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest`).
- **Auth**: usa `aws ecr get-login-password` + docker login.

---

## 8) IAM Roles for Tasks

- Cada task puede asumir un **IAM Role** diferente.
- Define qué servicios AWS puede acceder tu contenedor (S3, DynamoDB, SQS, etc.).
- **No hardcodear credenciales** nunca.
- Se configura en la Task Definition (`taskRoleArn`).

---

## 9) Integraciones clave

| Servicio | Para qué |
|---|---|
| **ALB/NLB** | Distribuir tráfico a tasks |
| **CloudWatch** | Logs, métricas, alarmas |
| **ECR** | Almacenar imágenes Docker |
| **IAM** | Roles y permisos por task |
| **Secrets Manager / SSM** | Inyectar secrets sin hardcodear |
| **CodeDeploy** | Despliegues Blue/Green |

---

## 10) Checklist rápido de examen

1. ¿La pregunta habla de **contenedores**? → ECS.
2. ¿Menciona **sin servidores / sin gestionar infra**? → Fargate.
3. ¿Necesita **red aislada por tarea**? → awsvpc network mode.
4. ¿Pide **escalado automático de tasks**? → Service Auto Scaling.
5. ¿Pide **zero downtime deployment**? → Blue/Green con CodeDeploy.
6. ¿Dice "almacenar imágenes Docker en AWS"? → ECR.
7. ¿El contenedor necesita acceder a S3/DynamoDB? → IAM Task Role.

---

## 11) AWS Copilot — CLI para ECS sin dolor

**Qué es**: herramienta CLI oficial de AWS para crear, publicar y operar aplicaciones en contenedores listas para producción, sin tener que configurar toda la infra a mano.

**Qué hace por vos**:
- Provisiona toda la infra necesaria automáticamente: ECS, VPC, ALB/ELB, ECR, IAM roles...
- Arma un **pipeline de despliegue** con CodePipeline integrado.
- Permite desplegar en **múltiples entornos** (dev, staging, prod) con un solo comando.
- Incluye herramientas de observabilidad: logs, estado de salud, troubleshooting.

**Dónde puede correr tus apps**:
- Amazon ECS
- AWS Fargate
- AWS App Runner

**Flujo de trabajo**:
```
Arquitectura (CLI/YAML)  →  AWS Copilot  →  infra bien diseñada
                                          →  pipeline de despliegue
                                          →  operaciones y troubleshooting
                                                    ↓
                                          ECS / Fargate / App Runner
```

**Truco examen**: Si la pregunta dice "desplegar contenedores sin configurar infraestructura manualmente" o "automatizar pipeline de ECS con un solo comando" → **AWS Copilot**.

---

## 12) Amazon EKS — Elastic Kubernetes Service

**Qué es**: servicio administrado de AWS para correr clústeres **Kubernetes** en la nube.

- Kubernetes = sistema open source para despliegue, escalado y gestión de contenedores (generalmente Docker).
- EKS es la **alternativa a ECS** con objetivo similar pero API y modelo diferente.
- **Kubernetes es agnóstico a la nube** → corre en AWS, Azure, GCP, on-premises. Por eso es la opción cuando ya tenés Kubernetes en otro lado y querés migrar a AWS sin reescribir nada.

### Launch types (igual que ECS)
- **EC2**: vos provisionás los nodos trabajadores (worker nodes).
- **Fargate**: contenedores sin servidor, AWS maneja la infra.

### ECS vs EKS — cuándo usar cada uno

| | **ECS** | **EKS** |
|---|---|---|
| Orquestador | Propio de AWS | Kubernetes (open source) |
| Curva de aprendizaje | Menor | Mayor |
| Portabilidad | Solo AWS | Multi-cloud / on-premises |
| Caso típico | Apps nuevas en AWS | Migración desde Kubernetes existente |

**Truco examen**:
- "Ya usamos Kubernetes on-premises y queremos migrar a AWS" → **EKS**
- "Queremos contenedores administrados en AWS sin experiencia previa en K8s" → **ECS/Fargate**
- "Necesitamos portabilidad entre nubes" → **EKS** (Kubernetes es agnóstico)

---

## 13) Tips finales

- **Task Definition es inmutable**: no se edita, se crea una nueva revisión.
- **Service usa una revisión específica** de la Task Definition.
- **Fargate tasks no comparten recursos** entre sí (aislamiento real).
- El **Application Load Balancer** es el más común con ECS (routing basado en path/host).
- **Service Discovery** (Cloud Map) permite que tasks se encuentren entre sí por nombre DNS.