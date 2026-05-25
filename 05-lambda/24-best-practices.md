# Lambda Best Practices

## 1. Trabajo pesado FUERA del handler

### ❌ Anti-patrón
```python
def lambda_handler(event, context):
    # Cada invocación paga estos costos
    db = connect_to_database()        # SLOW! 
    s3_client = boto3.client('s3')    # SLOW!
    secrets = get_secrets_from_ssm()  # SLOW!
    
    return process_event(event)
```

### ✅ Patrón correcto
```python
# Una sola vez por container (container reuse)
db = connect_to_database()        # Fuera del handler
s3_client = boto3.client('s3')    # Fuera del handler  
secrets = get_secrets_from_ssm()  # Fuera del handler

def lambda_handler(event, context):
    # Solo la lógica de negocio
    return process_event(event)     # FAST!
```

### ¿Por qué funciona?
- **Container reuse**: AWS reutiliza containers entre invocaciones
- **Warm starts**: El código fuera del handler ya está inicializado
- **Cold starts**: Solo paga la inicialización una vez

---

## 2. Variables de entorno para configuración

### ❌ Anti-patrón - Hardcodeo
```python
def lambda_handler(event, context):
    bucket = "my-prod-bucket-123"           # NO!
    db_url = "prod.rds.amazonaws.com"       # NO!
    api_key = "sk-abc123..."                # NUNCA!
```

### ✅ Patrón correcto
```python
import os

# Variables de entorno (no sensibles)
DB_URL = os.environ['DB_CONNECTION_STRING']
S3_BUCKET = os.environ['S3_BUCKET_NAME']
REGION = os.environ['AWS_REGION']

# Secretos cifrados con KMS
API_KEY = decrypt_kms(os.environ['API_KEY_ENCRYPTED'])

def lambda_handler(event, context):
    # Usar variables...
```

### Configuración de variables:
```json
{
  "Environment": {
    "Variables": {
      "DB_CONNECTION_STRING": "prod-db.cluster.amazonaws.com",
      "S3_BUCKET_NAME": "my-app-bucket", 
      "API_KEY_ENCRYPTED": "AQICAHh+..."
    }
  }
}
```

---

## 3. Minimizar deployment package

### ✅ Package optimizado
```
my-lambda/
├─ handler.py              # Solo código necesario
├─ utils/                  # Helpers específicos
│   └─ validators.py
├─ requirements.txt        # Solo deps usadas
└─ data/                   # Solo datos necesarios
    └─ config.json
```

### ❌ Package hinchado  
```
my-lambda/
├─ handler.py
├─ tests/                  # NO incluir tests
├─ .git/                   # NO incluir git
├─ node_modules/           # TODO node_modules
├─ __pycache__/           # NO incluir cache
├─ .env                   # NO incluir secrets  
└─ docs/                  # NO incluir docs
```

### Estrategias de optimización:

**Para dependencias grandes:**
```bash
# Solo production dependencies
npm install --production

# Python: solo packages necesarios
pip install --target ./package requests boto3
```

**Para código compartido:**
```yaml
# Usar Lambda Layers
Layers:
  - arn:aws:lambda:region:account:layer:shared-utils:1
  - arn:aws:lambda:region:account:layer:common-deps:2
```

**Para packages > 250MB:**
```dockerfile
# Container Images
FROM public.ecr.aws/lambda/python:3.9
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["app.lambda_handler"]
```

---

## 4. NO recursión infinita

### ❌ PELIGROSO - Loop infinito
```python
import boto3

def lambda_handler(event, context):
    lambda_client = boto3.client('lambda')
    
    if some_condition:
        # Lambda se llama a sí misma = LOOP INFINITO!
        lambda_client.invoke(
            FunctionName=context.function_name,
            Payload=json.dumps(event)
        )
```

### ✅ Alternativas seguras

**Step Functions:**
```python
def lambda_handler(event, context):
    step_functions = boto3.client('stepfunctions')
    
    step_functions.start_execution(
        stateMachineArn='arn:aws:states:...',
        input=json.dumps(event)
    )
```

**SQS para retry:**
```python
def lambda_handler(event, context):
    try:
        process_event(event)
    except Exception as e:
        # Enviar a DLQ para retry manual
        sqs.send_message(
            QueueUrl=DLQ_URL,
            MessageBody=json.dumps(event)
        )
```

**Contador de reintentos:**
```python
def lambda_handler(event, context):
    retry_count = event.get('retry_count', 0)
    
    if retry_count > MAX_RETRIES:
        return {'status': 'failed', 'reason': 'max_retries_exceeded'}
    
    try:
        return process_event(event)
    except Exception as e:
        # Incrementar contador y reintentar
        event['retry_count'] = retry_count + 1
        invoke_lambda_delayed(event)
```

---

## Otras Best Practices importantes

### Memory allocation
```python
# Más memoria = más CPU = menos tiempo = menos costo
# Para workloads CPU-intensive: usar más memoria
```

### Error handling
```python
def lambda_handler(event, context):
    try:
        return process_event(event)
    except SpecificError as e:
        # Log específico + retry
        logger.error(f"Specific error: {e}")
        raise
    except Exception as e:
        # Log genérico + fail
        logger.error(f"Unexpected error: {e}")
        return {'statusCode': 500, 'body': 'Internal error'}
```

### Logging structured
```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(json.dumps({
        'event': 'function_start',
        'request_id': context.aws_request_id,
        'remaining_time': context.get_remaining_time_in_millis()
    }))
```

---

## Para el examen

### Preguntas típicas:

**"Optimizar cold starts"**
→ Conexiones DB y SDK clients fuera del handler

**"Evitar hardcodeo de configuración"**  
→ Variables de entorno + KMS para secretos

**"Lambda deployment muy lento"**
→ Minimizar package size, usar Layers

**"Factura Lambda muy alta"**
→ Check recursión infinita, optimizar memoria/tiempo

**"Lambda falla inconsistentemente"**  
→ Verificar container reuse assumptions, proper error handling