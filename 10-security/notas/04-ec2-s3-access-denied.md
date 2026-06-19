# EC2 → S3 "Access Denied" — qué chequear

S3 no vive en una VPC. Es un servicio regional con endpoint público (o VPC Endpoint privado).

## Las 2 capas a validar siempre

| Capa                   | Qué chequear                                            |
| ---------------------- | ------------------------------------------------------- |
| **IAM Role de la EC2** | ¿Tiene `s3:GetObject` / `s3:PutObject` sobre el bucket? |
| **Bucket Policy**      | ¿Permite al rol/cuenta acceder?                         |

> AWS usa **permissions intersection**: ambas capas deben permitir. Una sola que niegue → Access Denied.

## Trampas típicas

| Opción que aparece        | Por qué descartarla                                      |
| ------------------------- | -------------------------------------------------------- |
| Security Groups (inbound) | EC2 hace requests **salientes** a S3. Inbound no aplica. |
| **VPC Peering**           | Conecta VPC↔VPC. **S3 no está en una VPC**. Irrelevante. |
| "Hacer bucket público"    | Anti-pattern. Descarte automático.                       |
| NACL inbound              | Lo mismo que SG inbound — outbound es lo que cuenta.     |

## "Privado" significa cosas distintas

- **VPC privada** = red aislada sin IP pública.
- **Bucket privado** = policy bloquea acceso público, pero el bucket vive en internet AWS.

EC2 en VPC privada → llega a S3 vía **NAT Gateway** (público) o **VPC Endpoint** (privado, recomendado).
