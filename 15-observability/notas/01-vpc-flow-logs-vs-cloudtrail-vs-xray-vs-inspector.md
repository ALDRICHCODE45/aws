# Observabilidad AWS — qué servicio para qué

| Servicio               | Qué registra                                               |
| ---------------------- | ---------------------------------------------------------- |
| **VPC Flow Logs**      | Tráfico **IP** en VPC/subnet/ENI                           |
| **CloudTrail**         | Llamadas a la **API de AWS** (auditoría de acciones)       |
| **AWS Inspector**      | **Vulnerabilidades** de seguridad (CVEs) en EC2/ECR/Lambda |
| **AWS X-Ray**          | **Trazas** entre microservicios (latencia por servicio)    |
| **CloudWatch Logs**    | Logs de aplicación                                         |
| **CloudWatch Metrics** | Métricas custom y de servicios                             |

## Disparadores

| Pregunta dice                                                    | Servicio          |
| ---------------------------------------------------------------- | ----------------- |
| "tráfico IP" / "paquetes de red" / "entrante/saliente" / "ENI"   | **VPC Flow Logs** |
| "quién hizo qué acción AWS" / "auditoría de API"                 | **CloudTrail**    |
| "vulnerabilidades" / "scanning" / "CVEs"                         | **Inspector**     |
| "trazar requests entre servicios" / "performance microservicios" | **X-Ray**         |
| "regulatorios" + "tráfico de red"                                | **VPC Flow Logs** |

## VPC Flow Logs — detalles

- Capturás a nivel **VPC / subnet / ENI**.
- Destino: **CloudWatch Logs / S3 / Kinesis Firehose**.
- Filtro: **ACCEPT / REJECT / ALL** (default ALL).
- NO captura: DNS Amazon, metadata service (169.254.169.254), DHCP.
