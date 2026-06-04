# DynamoDB — Seguridad y otras características

---

## Seguridad

- **VPC Endpoints** → acceder a DynamoDB sin salir a internet
- **IAM** → controla todo el acceso
- **Cifrado en reposo** → AWS KMS
- **Cifrado en tránsito** → SSL/TLS

## Backups

- **PITR (Point-In-Time Recovery)** → restaurar a cualquier segundo en los últimos **35 días**
- Sin impacto en rendimiento

## Global Tables

- Multi-región, multi-activa, totalmente replicada
- Recuerda: siempre **eventually consistent** entre regiones

## DynamoDB Local

- DynamoDB corriendo en tu máquina (Docker) para desarrollo/testing sin internet
- No relevante para el examen

## Migración

- **AWS DMS** (Database Migration Service) → migrar desde MongoDB, Oracle, MySQL, S3 hacia DynamoDB
