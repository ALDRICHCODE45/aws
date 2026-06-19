# ElastiCache — Memcached vs Redis

| | Memcached | Redis |
| -- | --------- | ----- |
| Threading | **Multi-thread** | Single-thread |
| Escalado horizontal | ✅ Auto (sharding) | Replicación (no auto) |
| Estructuras | Solo strings/objetos simples | Listas, sets, hashes, sorted sets |
| Persistencia | ❌ | ✅ Sí |
| Pub/Sub | ❌ | ✅ |
| Multi-AZ / Failover | ❌ | ✅ |
| Backup/Restore | ❌ | ✅ |
| Transactions | ❌ | ✅ |
| Encryption + Auth | Básico | Completo (TLS) |

## Disparadores

| Pregunta dice | Servicio |
| ------------- | -------- |
| "multi-hilo" / "múltiples cores" | **Memcached** |
| "escalar horizontal auto" + cache simple | **Memcached** |
| "objetos simples" | **Memcached** |
| "leaderboard" / "ranking" / "sorted set" | **Redis** |
| "persistencia" / "backup" | **Redis** |
| "pub/sub" | **Redis** |
| "alta disponibilidad / multi-AZ" | **Redis** |
| "transacciones" | **Redis** |

## Regla mental rápida

Cache **simple, multi-hilo, escala plana** → **Memcached**.
**Cualquier feature avanzado** (persistencia, HA, pub/sub, estructuras) → **Redis**.

## Sesiones de usuario distribuidas

| Solución | Veredicto |
| -------- | --------- |
| **ElastiCache Redis** | ✅ Estándar |
| **DynamoDB** (con TTL) | ✅ Alternativa válida |
| ElastiCache Memcached | ⚠️ Solo sin requerir HA |
| Sticky Sessions | ❌ Ata user a una EC2 (anti-HA) |
| Sesiones en memoria del servidor | ❌ Anti-escalable |
| CloudFront | ❌ No es server-side state |
