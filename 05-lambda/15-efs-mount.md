# Lambda — Montaje de EFS

Resumen corto para examen.

---

## Idea principal

**AWS Lambda** puede montar un **Elastic File System (EFS)** como storage compartido entre múltiples funciones (y otros servicios).
A diferencia de `/tmp`, EFS persiste y se comparte entre contenedores y Lambdas distintas.

---

## Requisitos

- Lambda debe estar **dentro de una Virtual Private Cloud (VPC)**.
- EFS y Lambda deben estar en la misma VPC.
- Sin VPC no hay acceso a EFS.

---

## Cómo se monta

Al configurar la integración, se define:

- **EFS Access Point** a usar.
- **Punto de montaje local** dentro de la Lambda, ej: `/mnt/data`.

Lambda hace el `mount` automáticamente en el cold start. El código accede como filesystem local:

```go
data, _ := os.ReadFile("/mnt/data/config.json")
```

---

## EFS Access Point

Puerta de entrada al filesystem con reglas predefinidas:

- Define un **directorio raíz virtual** dentro del EFS.
- Define un **usuario POSIX** (UID/GID) con el que la Lambda accede.

Sirve para **aislar Lambdas** dentro del mismo EFS:

```
EFS real:
├── /app1/    → Access Point 1 (root: /app1, UID 1000) → Lambda A
├── /app2/    → Access Point 2 (root: /app2, UID 2000) → Lambda B
└── /shared/  → Access Point 3 (root: /shared)
```

Cada Lambda solo ve su subárbol y nada más.

---

## Diagrama típico

```
┌────────────── VPC ──────────────┐
│  Subred AZ A      Subred AZ B   │
│   λ   λ            λ   λ        │
│    └───┴────┬───────┴───┘       │
│             │                   │
│      EFS Access Point           │
│             │                   │
│        EFS File System          │
└─────────────────────────────────┘
```

EFS es multi-AZ por naturaleza → las Lambdas de varias AZ ven el mismo storage.

---

## Límites importantes para examen

- **Una instancia de Lambda = una conexión TCP a EFS**.
- Si la Lambda escala mucho (miles concurrentes), abre muchas conexiones.
- EFS tiene **límite de ráfaga de conexiones** (burst).
- Lambdas con escalado masivo y rápido pueden saturar EFS → preferir **Amazon S3** en esos casos.

---

## Cuándo usar EFS con Lambda

- Archivos grandes que varias Lambdas comparten.
- Cargar modelos de Machine Learning sin descargarlos en cada cold start.
- Compartir estado con otros servicios (EC2, ECS) que montan el mismo EFS.
- Aplicaciones legacy que esperan rutas POSIX.

---

## Comparativa de storage para Lambda

| Necesidad | Storage |
|---|---|
| Temporal del mismo contenedor | `/tmp` |
| Compartido POSIX entre Lambdas | **EFS** |
| Compartido masivo, no POSIX | **Amazon S3** |
| Clave-valor compartida | **Amazon DynamoDB** |
| Cache baja latencia | **Amazon ElastiCache** |

---

## Punto de examen

- EFS con Lambda requiere VPC.
- Usar Access Points para aislar y controlar permisos POSIX.
- Cuidado con límite de conexiones en escalado masivo.
- Multi-AZ por defecto → alta disponibilidad.
