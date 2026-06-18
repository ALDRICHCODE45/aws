# SAM y herramientas Code* — qué usa para qué

## Disparadores → herramienta

| Pregunta menciona | Herramienta |
| ----------------- | ----------- |
| Serverless + "localmente" + simular/depurar | **SAM** |
| Infra serverless en YAML simplificado | **SAM** |
| Infra cualquiera con código (Python/TS/Go) | **CDK** |
| Infra cualquiera en YAML/JSON declarativo | **CloudFormation** |
| Desplegar código ya construido | **CodeDeploy** |
| Pipeline build → test → deploy | **CodePipeline** |
| Compilar/empaquetar código | **CodeBuild** |
| Repo Git en AWS | **CodeCommit** |
| Plugin de IDE | **AWS Toolkit** (NO para CI/CD) |
| Servidores con Chef/Puppet | **OpsWorks** (legacy) |

## Transform — obligatorio para SAM

```yaml
Transform: AWS::Serverless-2016-10-31   # OBLIGATORIO si usás SAM
Resources:                              # única SIEMPRE obligatoria
```

`AWSTemplateFormatVersion` suena obligatoria pero **NO lo es** (trampa).

## Trampas

1. **Especialización gana**: si la pregunta es serverless → SAM, no CloudFormation (aunque CFN también funcionaría).
2. **Anclaje**: no te ancles en UNA palabra del enunciado. La opción correcta cubre TODAS.
