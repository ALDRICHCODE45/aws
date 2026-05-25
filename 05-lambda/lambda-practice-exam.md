# Lambda Practice Exam - DVA-C02 Style

**Instrucciones:** 20 preguntas tipo examen AWS. Múltiple choice. Tiempo recomendado: 30 minutos.

---

## Pregunta 1
Una función Lambda recibe eventos de S3 y procesa archivos de hasta 2GB. El procesamiento falla frecuentemente con "Task timed out". ¿Cuál es la mejor solución?

A) Aumentar la memoria de 512MB a 1GB  
B) Cambiar el timeout de 3 minutos a 15 minutos  
C) Usar Step Functions para dividir el procesamiento  
D) Cambiar a Container Images  

---

## Pregunta 2
Tu función Lambda necesita conectarse a una base de datos RDS en una VPC privada. ¿Qué configuración es necesaria?

A) Solo configurar VPC subnets en la función Lambda  
B) VPC subnets + Security Groups + NAT Gateway para internet  
C) VPC subnets + Security Groups solamente  
D) Usar VPC Endpoints para RDS  

---

## Pregunta 3
Una función Lambda tarda 2 segundos en cold start porque inicializa conexiones a DynamoDB. ¿Cómo optimizar?

A) Usar Provisioned Concurrency  
B) Mover las conexiones fuera del handler  
C) Aumentar la memoria asignada  
D) Usar Reserved Concurrency  

---

## Pregunta 4
Tu aplicación necesita deployment gradual de Lambda con rollback automático si hay errores. ¿Qué solución usar?

A) Lambda Aliases con weighted routing manual  
B) CodeDeploy con Canary deployment + hooks  
C) Lambda Versions con blue/green manual  
D) CloudFormation con rollback automático  

---

## Pregunta 5
Una función Lambda procesa 1000 eventos por segundo de SQS. Durante picos de tráfico aparecen "429 TooManyRequests". ¿Cuál es la causa más probable?

A) El límite de concurrencia regional (1000) se alcanzó  
B) SQS tiene throttling  
C) Lambda no tiene Reserved Concurrency configurado  
D) El batch size es muy grande  

---

## Pregunta 6
Tu función Lambda necesita acceso público vía HTTPS sin usar API Gateway. ¿Qué opción usar?

A) Lambda Function URLs con AuthType NONE  
B) Application Load Balancer targeting Lambda  
C) CloudFront + Lambda@Edge  
D) Lambda Function URLs con AuthType AWS_IAM  

---

## Pregunta 7
Una función Lambda de 180MB (descomprimida) falla al deployar con "ValidationException". ¿Cuál puede ser el problema?

A) Supera el límite de 250MB descomprimido  
B) Supera el límite de 50MB comprimido  
C) Necesita usar S3 para deployment  
D) Necesita usar Container Images  

---

## Pregunta 8
Necesitas que múltiples cuentas AWS invoquen tu Lambda con Function URL. ¿Qué configuración usar?

A) AuthType NONE  
B) AuthType AWS_IAM + resource-based policy + identity-based policy  
C) AuthType AWS_IAM + solo resource-based policy  
D) AuthType AWS_IAM + solo identity-based policy  

---

## Pregunta 9
Tu Lambda procesa archivos de S3 pero a veces recibe el mismo evento duplicado. ¿Cómo manejar la idempotencia?

A) Configurar S3 para enviar eventos una sola vez  
B) Usar DynamoDB para trackear eventos procesados  
C) Configurar Dead Letter Queue  
D) Usar SQS FIFO en lugar de S3 events  

---

## Pregunta 10
Una función Lambda@Edge falla con "The runtime parameter of python3.9 is not supported". ¿Cuál es el problema?

A) Lambda@Edge no soporta Python 3.9  
B) Falta configuración de CloudFront  
C) Lambda@Edge solo funciona en us-east-1  
D) Necesita usar Container Images  

---

## Pregunta 11
Tu función Lambda usa variables de entorno para credenciales de API. ¿Cuál es la mejor práctica de seguridad?

A) Usar variables de entorno normales  
B) Cifrar con KMS y decrypt en tiempo de ejecución  
C) Guardar en S3 y descargar al inicio  
D) Hardcodear en el código  

---

## Pregunta 12
Una función Lambda procesa eventos de Kinesis. Durante errores, quieres enviar registros fallidos a SQS. ¿Qué configurar?

A) Dead Letter Queue en Kinesis  
B) Destinations para async invocation  
C) Error handling en Event Source Mapping  
D) Try/catch en el código + SQS send  

---

## Pregunta 13
Necesitas debuggear performance de una función Lambda que tiene múltiples métodos internos lentos. ¿Qué herramienta usar?

A) CloudWatch Logs  
B) X-Ray tracing  
C) CodeGuru Profiler  
D) CloudWatch Metrics  

---

## Pregunta 14
Tu función Lambda necesita 8GB de dependencias ML. El deployment falla. ¿Cuál es la mejor solución?

A) Usar Lambda Layers de 8GB  
B) Usar Container Images  
C) Dividir en múltiples funciones  
D) Usar S3 deployment + download en runtime  

---

## Pregunta 15
Una función Lambda debe procesar archivos grandes usando /tmp directory. ¿Cuál es el límite configurable de /tmp?

A) Fijo en 512MB  
B) 512MB a 1GB  
C) 512MB a 10GB  
D) 1GB a 10GB  

---

## Pregunta 16
Tu Lambda recibe invocaciones síncronas de API Gateway. Durante fallos, quieres retry automático. ¿Cómo configurar?

A) Dead Letter Queue en Lambda  
B) Destinations en Lambda  
C) Error handling en API Gateway  
D) Las invocaciones síncronas no permiten retry automático  

---

## Pregunta 17
Una función Lambda cuesta mucho porque se ejecuta 15 minutos. ¿Cuál es la primera optimización a considerar?

A) Usar Step Functions para dividir el trabajo  
B) Aumentar la memoria para reducir tiempo  
C) Cambiar a EC2  
D) Usar Reserved Concurrency  

---

## Pregunta 18
Tu función Lambda necesita diferentes comportamientos para DEV, TEST y PROD usando el mismo código. ¿Cuál es la mejor práctica?

A) Usar versiones diferentes de Lambda  
B) Usar aliases apuntando a versiones + variables de entorno  
C) Duplicar la función para cada ambiente  
D) Usar IF statements basados en región  

---

## Pregunta 19
Una función Lambda procesa eventos de DynamoDB Streams. ¿Cuál es el paralelism factor máximo?

A) 1 (procesamiento secuencial por partition key)  
B) 10 (configuración de batch size)  
C) 100 (límite de Event Source Mapping)  
D) 1000 (límite de concurrencia)  

---

## Pregunta 20
Tu función Lambda se llama a sí misma recursivamente y generas una factura de $5000. ¿Cómo prevenir esto en el futuro?

A) Configurar Reserved Concurrency en 0  
B) Configurar timeout más bajo  
C) Usar Step Functions en lugar de recursión  
D) Configurar Dead Letter Queue  

---

## Respuestas Correctas

1. **C** - Step Functions (archivos 2GB + timeout 15min = dividir trabajo)
2. **B** - VPC subnets + SG + NAT (para internet access)
3. **B** - Mover conexiones fuera del handler (container reuse)
4. **B** - CodeDeploy Canary + hooks (rollback automático)
5. **A** - Límite concurrencia regional alcanzado
6. **A** - Function URLs AuthType NONE (acceso público)
7. **B** - ZIP > 50MB comprimido (usar S3)
8. **B** - Cross-account = ambas políticas needed
9. **B** - DynamoDB para idempotency tracking
10. **A** - Lambda@Edge no soporta Python 3.9
11. **B** - KMS encryption para secretos
12. **C** - Event Source Mapping error handling
13. **C** - CodeGuru para performance interno
14. **B** - Container Images para 8GB
15. **C** - /tmp: 512MB a 10GB configurable
16. **D** - Sync invocations = no retry automático
17. **A** - 15min = dividir con Step Functions
18. **B** - Aliases + variables de entorno
19. **A** - DynamoDB Streams = secuencial por partition
20. **C** - Step Functions previene recursión infinita