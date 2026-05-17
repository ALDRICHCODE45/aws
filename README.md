# AWS Developer Associate — Plan de estudio (rápido + sólido)

Bienvenido loco. Esta carpeta va a ser tu **base de entrenamiento real** para preparar la certificación de AWS Developer Associate.

## Objetivo

Avanzar rápido, pero con fundamentos fuertes:

- **Concepto claro** (qué es y por qué existe)
- **Mental model** (cómo pensarlo en arquitectura)
- **Aplicación práctica** (ejercicios reales)
- **Criterio de examen** (trampas y diferencias clave)

## Estructura

- `00-roadmap/`
  - Plan por semanas, método de estudio y cómo medir progreso.
- `01-foundations/`
  - Fundamentos de infraestructura global (regiones, AZ, edge locations).
  - IAM básico (`03-iam-basics.md`): usuarios, grupos, políticas y errores de examen.
  - IAM security tools (`04-iam-security-tools.md`): Credential Report vs Access Advisor.
  - EC2 basics, Security Groups, Purchasing Options (`05`, `06`, `07`).
  - VPC Networking — IGW + NACL + flujo de tráfico (`08-vpc-networking-igw-nacl.md`).
  - EC2 Storage Deep Dive (`09-ec2-storage-deep-dive.md`): EBS/AMI/Instance Store/EFS, Multi-Attach y trampas de examen.
   - ECS — Elastic Container Service (`10-ecs-container-service.md`): Clusters, Services, Task Definitions, Fargate vs EC2, despliegues, ECR.
   - Cognito — Identity & Authentication (`11-cognito-identity.md`): User Pools, Identity Pools, tokens, triggers, integración con DynamoDB.
   - Lambda — índice (`12-lambda.md`): apunta a la carpeta `05-lambda/`.
- `02-labs/`
  - Ejercicios prácticos y mini-proyectos con AWS CLI/IaC/código.
- `04-architectures/`
  - Patrones de arquitectura típicos de AWS con foco en examen. Cada arquitectura en su propio archivo.
  - Web App 3 niveles (`01-3-tier-web-app.md`): ELB + EC2 + ElastiCache + RDS, Multi-AZ, Auto Scaling.
- `05-lambda/`
  - Apuntes de Lambda separados por sub-tema. Archivos chicos para repaso rápido.
  - Fundamentos, ALB sync, async + idempotencia, EventBridge (más por venir).
- `03-exam/`
  - Resúmenes de alto impacto, preguntas tipo examen y errores frecuentes.
  - Incluye: `regions-az-edge-repaso-practico.md` para repaso rápido del bloque 1.

## Cómo usar este repo (sugerido)

1. Leé un tema en `01-foundations`.
2. Respondé el checkpoint de memoria **sin mirar apuntes**.
3. Hacé el lab correspondiente en `02-labs`.
4. Cerrá con repaso de examen en `03-exam`.

Si en algún punto memorizás sin entender, frenamos y reconstruimos el concepto. **CONCEPTO > RECETA**.
