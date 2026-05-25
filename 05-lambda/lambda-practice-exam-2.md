# Lambda Practice Exam #2 - DVA-C02 Style

**Instrucciones:** 15 preguntas tipo examen AWS. Múltiple choice. Tiempo recomendado: 20 minutos.

---

## Pregunta 1
Tu función Lambda en CloudFormation usa S3 para deployment. Actualizaste el código en S3, pero cuando deployás la stack, Lambda sigue usando código anterior. ¿Cuál es la causa más probable?

A) Falta actualizar S3Bucket en CloudFormation  
B) Falta actualizar S3Key en CloudFormation  
C) Falta actualizar S3ObjectVersion en CloudFormation  
D) CloudFormation tiene caché de S3 objects  

---

## Pregunta 2
Una función Lambda necesita acceso cross-account. La cuenta B quiere invocar una función en cuenta A. ¿Qué configuración es necesaria?

A) Solo Execution Role en cuenta A  
B) Solo Resource-based Policy en función Lambda de cuenta A  
C) Resource-based Policy en función Lambda + Identity-based Policy en cuenta B  
D) Cross-account IAM Role compartido  

---

## Pregunta 3
Tu función Lambda procesa eventos de una cola SQS FIFO con 50 mensajes. ¿Cuál es el máximo batch size que Lambda puede procesar por invocación desde SQS?

A) 1 mensaje  
B) 10 mensajes  
C) 50 mensajes  
D) 100 mensajes  

---

## Pregunta 4
Una función Lambda falla y necesitas enviar mensajes fallidos a una cola SQS. Tu función procesa eventos de Kinesis Data Streams. ¿Dónde configurás error handling?

A) Dead Letter Queue en Lambda function  
B) Destinations en Lambda function  
C) Event Source Mapping failure configuration  
D) Kinesis Stream error handling  

---

## Pregunta 5
Tu función Lambda necesita almacenar un token de autenticación de 6KB. Variables de entorno fallan con error de tamaño. ¿Cuál es la mejor alternativa?

A) Dividir el token en múltiples variables de entorno  
B) Usar Systems Manager Parameter Store  
C) Hardcodear el token en el código  
D) Almacenar en DynamoDB y consultarlo en runtime  

---

## Pregunta 6
Una función Lambda procesa archivos grandes usando /tmp directory. Necesitas 8GB de espacio temporal. ¿Es esto posible?

A) No, /tmp está limitado a 512MB fijo  
B) No, /tmp máximo es 1GB  
C) Sí, /tmp es configurable hasta 10GB  
D) Sí, pero solo con Container Images  

---

## Pregunta 7
Tu función Lambda usa Node.js con dependencia `aws-sdk` y `lodash`. ¿Cómo incluís las dependencias en el deployment?

A) Subir solo el código, AWS instala dependencias automáticamente  
B) Usar Lambda Layers separadas para cada dependencia  
C) Incluir node_modules en la misma carpeta que tu código  
D) Subir dependencias a S3 por separado  

---

## Pregunta 8
Una función Lambda configurada con Dead Letter Queue hacia SNS no envía mensajes fallidos. CloudWatch muestra errors pero DLQ vacía. ¿Cuál es la causa más probable?

A) SNS topic no existe  
B) DLQ mal configurada  
C) Lambda Execution Role sin permisos sns:Publish  
D) Lambda function no tiene retry configurado  

---

## Pregunta 9
¿Cuál de estos servicios NO requiere Event Source Mapping para invocar Lambda?

A) Amazon SQS Standard  
B) Amazon Kinesis Data Streams  
C) Amazon S3 Events  
D) DynamoDB Streams  

---

## Pregunta 10
Tu función Lambda@Edge falla con error "Runtime python3.9 not supported". ¿Cuál es el problema?

A) Lambda@Edge solo funciona en us-east-1  
B) Lambda@Edge no soporta Python 3.9  
C) Falta configurar CloudFront trigger  
D) Lambda@Edge requiere Container Images  

---

## Pregunta 11
Una función Lambda tarda 12 segundos en cold start. El código conecta a PostgreSQL RDS al inicio del handler. ¿Cómo optimizar?

A) Usar Provisioned Concurrency  
B) Mover conexión DB fuera del handler  
C) Aumentar memoria de Lambda  
D) Configurar Keep-Alive en RDS  

---

## Pregunta 12
Tu función Lambda procesa 2000 requests/segundo y recibes errores 429 TooManyRequests. El límite de concurrencia regional es 1000. ¿Cuál es la mejor solución inmediata?

A) Solicitar aumento de límite a AWS Support  
B) Configurar Reserved Concurrency en 500  
C) Dividir en múltiples funciones  
D) Usar SQS como buffer  

---

## Pregunta 13
Una función Lambda necesita invocar otra función Lambda de manera síncrona y obtener el resultado. ¿Cuál es la forma correcta?

A) boto3.client('lambda').invoke() con InvocationType='Event'  
B) boto3.client('lambda').invoke() con InvocationType='RequestResponse'  
C) SNS para comunicación entre Lambdas  
D) SQS para enviar mensajes entre Lambdas  

---

## Pregunta 14
Tu función Lambda usa una Layer compartida entre múltiples funciones. La Layer contiene dependencias que cambian frecuentemente. ¿Cuál es la mejor práctica?

A) Actualizar la Layer existente  
B) Crear nueva versión de la Layer  
C) Eliminar Layer y usar deployment packages individuales  
D) Usar Layer mutable compartida  

---

## Pregunta 15
Una función Lambda procesa eventos de DynamoDB Streams pero necesita procesamiento estrictamente secuencial por partition key. ¿Cuál es la configuración correcta?

A) Parallelization factor = 10  
B) Parallelization factor = 1  
C) Batch size = 1  
D) Event Source Mapping concurrency = 1  

---

## Respuestas Correctas

1. **C** - S3ObjectVersion debe actualizarse para nuevo código
2. **C** - Cross-account requiere ambas políticas
3. **B** - SQS batch size máximo para Lambda es 10
4. **C** - Kinesis usa Event Source Mapping para error handling
5. **B** - Parameter Store para datos > 4KB
6. **C** - /tmp configurable hasta 10GB
7. **C** - node_modules incluido en deployment package
8. **C** - Execution Role necesita permisos SNS
9. **C** - S3 Events usa push model directo
10. **B** - Lambda@Edge limitado a Python 3.7/3.8
11. **B** - Conexiones fuera del handler para container reuse
12. **A** - Límite regional requiere solicitud de aumento
13. **B** - RequestResponse para invocación síncrona
14. **B** - Layers son immutable, crear nuevas versiones
15. **B** - Parallelization factor 1 para orden por partition