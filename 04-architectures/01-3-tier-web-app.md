# Arquitectura Típica: Web App de 3 Niveles (3-Tier)

Patrón de arquitectura clásico en AWS. **Alta probabilidad en el examen DVA-C02.**

---

## 1) Visión general

Separar responsabilidades en 3 capas, cada una en su propia subred con su propio nivel de acceso:

```
Internet
  → Route 53       (DNS)
  → ELB            (subred pública,  Multi-AZ)
  → EC2s           (subred privada,  Auto Scaling)
  → ElastiCache    (subred de datos, caché + sesiones)
  → RDS            (subred de datos, persistencia)
```

---

## 2) Nivel 1 — Subred Pública (lo que ve internet)

- **Route 53** resuelve el dominio y apunta al ELB.
- **ELB (Elastic Load Balancer)** es el único punto de entrada desde internet:
  - Multi-AZ → si cae una AZ, sigue funcionando.
  - Distribuye tráfico entre las EC2s del nivel 2.
  - Es el único recurso con IP pública expuesta.

**Security Group del ELB**: acepta 80/443 desde `0.0.0.0/0`.
**Security Group de las EC2s**: acepta tráfico solo desde el SG del ELB (nunca desde internet directo).

---

## 3) Nivel 2 — Subred Privada (lógica de negocio)

- **EC2s** corren tu aplicación.
- Distribuidas en un **Auto Scaling Group** en múltiples AZs.
- Sin IP pública → internet no puede llegar directo.
- Solo reciben tráfico del ELB.

**Auto Scaling**:
- Tráfico sube → lanza más EC2s automáticamente.
- Tráfico baja → las termina para ahorrar costo.

---

## 4) Nivel 3 — Subred de Datos (persistencia)

Dos servicios con propósitos distintos:

### ElastiCache (Redis / Memcached)
- **Sesiones de usuario**: centraliza el estado de sesión → cualquier EC2 puede atender cualquier request sin sticky sessions.
- **Caché de queries**: si el dato ya fue consultado, se devuelve desde memoria sin ir a la DB.
- Ultra rápido (in-memory).

### Amazon RDS
- Base de datos relacional persistente.
- **Read Replicas**: las EC2s leen de réplicas → no saturan el nodo primario.
- **Multi-AZ**: alta disponibilidad con failover automático.

---

## 5) Flujo completo de un request

```
Usuario
  → Route 53              (resuelve dominio)
  → ELB                   (distribuye carga entre EC2s)
  → EC2                   (procesa la lógica)
  → ElastiCache           (¿tengo esto en caché? → devuelvo directo)
  → RDS                   (si no está en caché → busco en DB → guardo en caché)
```

---

## 6) Por qué este diseño es robusto

| Problema | Solución |
|---|---|
| Una AZ cae | ELB + Auto Scaling Multi-AZ → otras AZs absorben |
| Pico de tráfico | Auto Scaling lanza más EC2s |
| Sesiones en múltiples servidores | ElastiCache centraliza el estado |
| DB saturada de lecturas | Read Replicas en RDS |
| Internet accede directo a DB | Subredes privadas + SGs restrictivos |

---

## 7) Checklist de examen

1. ¿Quién es el único punto de entrada desde internet? → **ELB**
2. ¿Las EC2s tienen IP pública? → **No**, están en subred privada.
3. ¿Cómo se resuelve el problema de sesiones entre múltiples EC2s? → **ElastiCache**
4. ¿Cómo se reduce la carga de lecturas en RDS? → **Read Replicas**
5. ¿Cómo se garantiza disponibilidad si cae una AZ? → **Multi-AZ en ELB + Auto Scaling + RDS**
6. ¿Qué SG tiene la EC2? → Solo acepta tráfico **desde el SG del ELB**, no desde internet.
