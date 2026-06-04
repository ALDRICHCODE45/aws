# DynamoDB — DAX (DynamoDB Accelerator)

Cache en memoria delante de DynamoDB. Resuelve Hot Keys sin cambiar código.

---

## Qué es

- Cache totalmente gestionado, específico para DynamoDB
- Latencia de **microsegundos** para lecturas en cache
- Compatible con las APIs existentes — no cambiás código
- Resuelve el problema de **Hot Key** (demasiadas lecturas a un mismo ítem)

## Arquitectura

- **Cluster** con múltiples **nodos** (servidores de cache)
- Más nodos = más capacidad + más disponibilidad
- Si un nodo cae, los otros siguen respondiendo

## Números para el examen

| Dato | Valor |
|---|---|
| TTL default del cache | **5 minutos** |
| Máximo nodos por cluster | **11** |
| Mínimo recomendado en producción | **3 nodos** (Multi-AZ) |

## Seguridad

Cifrado en reposo con KMS, vive dentro de la VPC, IAM, CloudTrail.

## DAX vs ElastiCache

- **DAX** → específico para DynamoDB, integrado, sin cambiar código
- **ElastiCache** → genérico, más código manual
- Para DynamoDB → DAX es casi siempre la respuesta correcta
