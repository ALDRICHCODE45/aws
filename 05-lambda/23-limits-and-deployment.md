# Lambda Limits - Por Región

## Límites de Ejecución

### Memory
- **Rango:** 128 MB - 10 GB (incrementos de 1 MB)
- CPU se asigna proporcionalmente a la memoria

### Timeout
- **Máximo:** 900 segundos (15 minutos)

### Variables de entorno
- **Tamaño total:** 4 KB

### Disk capacity (/tmp)
- **Rango:** 512 MB - 10 GB

### Concurrencia
- **Por región:** 1000 ejecuciones concurrentes (puede aumentarse)

---

## Límites de Deployment

### ZIP Files (método tradicional)

**Tamaño ZIP comprimido: 50 MB**
- El archivo `.zip` que subís a Lambda
- Límite para upload directo via consola/CLI

**Tamaño descomprimido: 250 MB**  
- Cuando AWS descomprime tu ZIP
- Total de todos los archivos (código + dependencias)

### ¿Por qué la diferencia?

Los archivos se comprimen según su tipo:
```
Node.js project:
├─ node_modules/ (muchos .js, .json) 
├─ Descomprimido: 200 MB
└─ ZIP: 35 MB (comprime ~82%)

Project con imágenes:
├─ Images/ (muchos .png, .jpg)
├─ Descomprimido: 200 MB  
└─ ZIP: 190 MB (comprime ~5%)
```

### Ejemplo práctico:
```
my-function.zip = 45 MB ✅ (comprimido, bajo límite)

Al descomprimir:
├─ app.py (2 MB)
├─ dependencies/ (200 MB) 
├─ data.json (30 MB)
├─ libs/ (15 MB)
└─ Total: 247 MB ✅ (bajo límite de 250 MB)
```

---

## ¿Qué hacer si superas los límites?

### ZIP > 50 MB (pero descomprimido < 250 MB)

**Problema:**
```bash
aws lambda update-function-code --zip-file fileb://big.zip
# ERROR: Deployment package size exceeds 50MB
```

**Solución:** Usar **S3** para deployment
```bash
# 1. Subir ZIP a S3
aws s3 cp big.zip s3://my-deployment-bucket/

# 2. Deploy desde S3  
aws lambda update-function-code \
  --function-name myFunc \
  --s3-bucket my-deployment-bucket \
  --s3-key big.zip
```

### Descomprimido > 250 MB

**Problema:**
```bash
# ERROR: Uncompressed size exceeds 250MB limit
```

**Solución:** **Container Images** (hasta 10 GB)
```dockerfile
FROM public.ecr.aws/lambda/python:3.9
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["app.lambda_handler"]
```

---

## Workaround: /tmp directory

**Uso del directorio /tmp:**
- Capacidad: **512 MB - 10 GB** (configurable)
- Puedes **cargar archivos adicionales** al inicio
- Persiste entre invocaciones del mismo container

### Ejemplo:
```python
import os
import boto3

def lambda_handler(event, context):
    # Descargar archivo pesado a /tmp al inicio
    if not os.path.exists('/tmp/large-model.bin'):
        s3 = boto3.client('s3')
        s3.download_file('my-bucket', 'large-model.bin', '/tmp/large-model.bin')
    
    # Usar archivo desde /tmp
    with open('/tmp/large-model.bin', 'rb') as f:
        # procesar...
```

---

## Para el examen

### Casos típicos:

**Pregunta:** "Tu función Node.js con muchas dependencias no se puede deployar"
```
Si ZIP comprimido > 50MB → Usar S3 para deployment
Si descomprimido > 250MB → Usar Container Images  
```

**Pregunta:** "Necesitás cargar un modelo ML de 500MB"
```
Container Images (hasta 10GB)
O usar /tmp directory + download desde S3
```

**Pregunta:** "Deploy falla: package too large"  
```
Verificar:
1. ¿ZIP > 50MB? → S3 deployment
2. ¿Descomprimido > 250MB? → Container Images
3. ¿Muchos archivos temporales? → /tmp directory
```

---

## Key Facts

- **ZIP comprimido:** 50 MB (upload directo)
- **ZIP descomprimido:** 250 MB (total archivos)  
- **Container Images:** 10 GB (alternativa)
- **S3 deployment:** Para ZIP > 50MB pero < 250MB descomprimido
- **Variables de entorno:** 4 KB total
- **Memory:** 128 MB - 10 GB
- **Timeout:** 15 minutos máximo
- **Concurrencia:** 1000/región (aumentable)