# Lambda — S3 Event Notifications

S3 puede disparar Lambda cuando ocurren operaciones en el bucket (upload, delete, etc).

---

## Flujo

```
Cliente sube archivo a S3
   ↓
S3 emite evento (notification configurada en el bucket)
   ↓ (asíncrono, at-least-once)
Lambda Function procesa el evento
```

Configuración: en el **bucket → Properties → Event notifications**.

---

## Casos de uso típicos
- Redimensionar imágenes al subir
- Antivirus scan de archivos subidos
- Indexar metadata en DynamoDB / OpenSearch
- Disparar pipelines de procesamiento
- Notificar a otros sistemas

---

## Destinos posibles (no solo Lambda)

| Destino | Uso |
|---|---|
| **Lambda** | Procesar el archivo |
| **SQS** (Standard) | Encolar para batch |
| **SNS** | Fan-out a múltiples suscriptores |
| **EventBridge** | Routing avanzado, filtros, **SQS FIFO**, cross-account |

> 🎯 **Examen**: S3 NO puede enviar directo a **SQS FIFO**. Para FIFO → S3 → EventBridge → SQS FIFO.

---

## Eventos comunes

| Evento | Cuándo |
|---|---|
| `s3:ObjectCreated:*` | Cualquier creación |
| `s3:ObjectCreated:Put` | Solo PUT |
| `s3:ObjectCreated:Post` | Solo POST |
| `s3:ObjectCreated:CompleteMultipartUpload` | Subida multipart completa |
| `s3:ObjectRemoved:*` | Borrados |
| `s3:ObjectRestore:*` | Restore desde Glacier |
| `s3:Replication:*` | Replicación |
| `s3:LifecycleExpiration:*` | Lifecycle expiration |

---

## Filtros disponibles

Podés filtrar qué objetos disparan la notification:
- **Prefix**: `images/`
- **Suffix**: `.jpg`

Ejemplo: solo disparar Lambda si el archivo está en `uploads/` y termina en `.png`.

---

## ⚠️ Loop infinito (trampa clásica)

Si la Lambda **escribe en el mismo bucket** que la disparó → **se invoca a sí misma infinitamente** → factura desastrosa.

**Soluciones**:
1. Usar **dos buckets** (input → output)
2. Usar **prefixes distintos** (`raw/` dispara, `processed/` no)

---

## Permisos

S3 necesita permiso para invocar Lambda → se configura como **resource-based policy** en la Lambda:

```json
{
  "Effect": "Allow",
  "Principal": { "Service": "s3.amazonaws.com" },
  "Action": "lambda:InvokeFunction",
  "Condition": {
    "StringEquals": { "AWS:SourceAccount": "123456789012" },
    "ArnLike": { "AWS:SourceArn": "arn:aws:s3:::mi-bucket" }
  }
}
```

La consola AWS lo configura automáticamente al crear el trigger.

---

## Características clave

- **Invocación asíncrona** → aplica todo lo de `03-async-invocation.md`
- **At-least-once delivery** → **idempotencia obligatoria**
- **Latencia**: segundos típicamente, hasta 1 min en algunos casos
- 100+ event notifications por bucket (límite del servicio)

---

## Checklist examen

- [ ] S3 → Lambda es **asíncrono** (idempotencia obligatoria)
- [ ] Destinos S3: **Lambda, SQS, SNS, EventBridge** (4 opciones)
- [ ] **NO directo a SQS FIFO** → usar EventBridge en el medio
- [ ] Filtros: **prefix** y **suffix**
- [ ] ⚠️ Lambda que escribe al mismo bucket = **loop infinito**
- [ ] Solución loop: 2 buckets o prefixes distintos
- [ ] Permisos: resource-based policy en la Lambda
- [ ] Latencia típica: segundos, hasta 1 min
- [ ] Eventos principales: `ObjectCreated:*`, `ObjectRemoved:*`, `Replication:*`, `LifecycleExpiration:*`
