# Lambda Container Images

## ¿Qué son?

Desde 2020, Lambda acepta **imágenes de Docker** hasta **10 GB** (además de ZIP files de 250 MB).

## ¿Cuándo usar Container Images?

- **Dependencias pesadas**: Librerías ML, procesamiento de imágenes que no caben en ZIP
- **Herramientas complejas**: ImageMagick, FFmpeg, bases de datos embebidas
- **Flujo unificado**: Mismo Dockerfile para ECS/Fargate y Lambda

## Arquitectura

```
┌─ DOCKER IMAGE (hasta 10GB) ────┐
│ ┌─ Código de aplicación ─────┐  │
│ ├─ Dependencias, datasets ──┤  │
│ └─ Base + Runtime API ──────┘  │
└────────────────────────────────┘
```

## Lambda Runtime API (clave del examen)

La imagen DEBE implementar la **Lambda Runtime API** que maneja:
- Recibir invocaciones
- Procesar eventos  
- Devolver respuestas

### Opciones:

1. **AWS Base Images** (recomendado): Ya implementan Runtime API
   - `public.ecr.aws/lambda/python:3.9`
   - `public.ecr.aws/lambda/nodejs:18`
   - etc.

2. **Custom Images**: Cualquier base, pero VOS implementás Runtime API

## Ejemplo Dockerfile

```dockerfile
FROM public.ecr.aws/lambda/python:3.9
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["app.lambda_handler"]
```

## Testing Local

Usar **Lambda Runtime Interface Emulator** para testear container localmente.

---

## Buenas Prácticas (examen)

### Optimización de imágenes:

**✅ Usar imágenes base AWS**
- Estables, construidas en Amazon Linux 2
- Almacenadas en caché por Lambda
- Arranque más rápido

**✅ Multi-stage builds**
```dockerfile
# Stage 1: Build dependencies
FROM python:3.9 as builder
COPY requirements.txt .
RUN pip install -r requirements.txt

# Stage 2: Final image
FROM public.ecr.aws/lambda/python:3.9
COPY --from=builder /app .
```

**✅ Orden de capas Docker**
- Lo **estable primero** (dependencias)
- Lo **que cambia frecuente al final** (código)

**✅ Repositorio único para funciones similares**
- ECR (Elastic Container Registry) compara capas
- Evita duplicados y reduce almacenamiento

### Casos de uso típicos:

- **Funciones Lambda grandes** (hasta 10 GB vs 250 MB ZIP)
- **Dependencias nativas complejas**
- **Reutilizar imágenes existentes** de otros servicios AWS

---

## Para el examen:

- Container Images = alternativa a ZIP cuando necesitás **más espacio**
- **Runtime API obligatoria** en la imagen
- **AWS Base Images** = opción más fácil y optimizada
- **Multi-stage builds** y **orden de capas** = optimización
- **ECR** almacena y optimiza capas duplicadas