# Lambda Final Exam - DVA-C02 Style

**Instrucciones:** 12 preguntas finales tipo examen AWS. Tiempo recomendado: 15 minutos.

---

## Pregunta 1
Una función Lambda procesa eventos de DynamoDB Streams para un sistema bancario. Los eventos deben procesarse en orden estricto para mantener balances correctos. ¿Cuál es la configuración correcta?

A) Batch size = 1  
B) Parallelization factor = 1  
C) Reserved Concurrency = 1  
D) Event Source Mapping concurrency = 1  

---

## Pregunta 2
Tu función Lambda usa Function URLs con AuthType AWS_IAM. Una cuenta externa (123456789012) no puede invocar tu función aunque tiene permisos IAM. ¿Qué falta configurar?

A) API Gateway integration  
B) CORS configuration  
C) Resource-based policy en tu función Lambda  
D) Lambda Execution Role permissions  

---

## Pregunta 3
Una función Lambda procesa eventos de SQS Standard queue con batch size 5. Durante picos de tráfico, algunos mensajes fallan y se reenvían múltiples veces. ¿Cuál es la mejor solución para evitar procesamiento duplicado?

A) Configurar DLQ en Lambda function  
B) Configurar DLQ en SQS queue  
C) Implementar idempotency en función Lambda  
D) Reducir batch size a 1  

---

## Pregunta 4
Tu función Lambda en CloudFormation usa inline code pero necesitas agregar dependencias que exceden 4KB. ¿Cuál es la transición correcta?

A) Usar Lambda Layers para dependencias  
B) Comprimir código inline con gzip  
C) Cambiar a S3 deployment con ZipFile  
D) Cambiar a Container Images  

---

## Pregunta 5
Una función Lambda@Edge debe modificar headers de respuesta para todos los usuarios globalmente. ¿En qué evento de CloudFront debe configurarse?

A) Viewer Request  
B) Origin Request  
C) Origin Response  
D) Viewer Response  

---

## Pregunta 6
Tu función Lambda tarda 3 segundos en procesar cada evento de Kinesis. Recibes mensajes de timeout aunque configuraste 15 minutos. ¿Cuál es el problema?

A) Kinesis timeout es más restrictivo que Lambda  
B) Event Source Mapping tiene timeout propio  
C) Batch size muy grande causa timeout  
D) Lambda timeout real es menor a 15 min  

---

## Pregunta 7
Una función Lambda debe invocar otra función Lambda y continuar procesamiento con el resultado. La segunda función puede tardar 2 minutos. ¿Cuál es el approach correcto?

A) InvocationType='Event' + polling del resultado  
B) InvocationType='RequestResponse' + timeout alto  
C) Step Functions para orchestration  
D) SQS entre las dos funciones  

---

## Pregunta 8
Tu función Lambda usa 1.5GB de memoria pero CloudWatch muestra solo 200MB utilizados. La función tarda 30 segundos en ejecutar. ¿Cuál es la optimización más efectiva?

A) Aumentar memoria a 3GB  
B) Reducir memoria a 512MB  
C) Usar Provisioned Concurrency  
D) Mover a Container Images  

---

## Pregunta 9
Una función Lambda necesita acceder a secretos de 10KB almacenados en Secrets Manager. ¿Dónde es más eficiente cachear estos secretos?

A) Variables de entorno  
B) /tmp directory  
C) Código fuera del handler  
D) DynamoDB table  

---

## Pregunta 10
Tu función Lambda procesa archivos de S3 de hasta 5GB. Lambda timeout está en 15 minutos pero sigue fallando. ¿Cuál es la mejor arquitectura?

A) Aumentar memoria para reducir tiempo  
B) Procesar archivos en chunks usando Step Functions  
C) Cambiar a ECS Fargate  
D) Usar EC2 para procesamiento pesado  

---

## Pregunta 11
Una función Lambda debe responder a API Gateway en menos de 29 segundos pero el procesamiento real puede tomar 10 minutos. ¿Cuál es el patrón correcto?

A) Aumentar API Gateway timeout  
B) Async pattern: responder immediately + procesar en background  
C) Usar WebSockets para long connections  
D) Polling desde frontend cada 30 segundos  

---

## Pregunta 12
Tu función Lambda usa CodeGuru Profiler y detecta que el método validatePayment() consume 80% del tiempo de ejecución. ¿Cuál es el siguiente paso?

A) Usar X-Ray para trace distribuido  
B) Optimizar específicamente validatePayment() method  
C) Aumentar memoria de Lambda  
D) Mover validatePayment a separate Lambda  

---

## Respuestas Correctas

1. **B** - Parallelization factor 1 para orden estricto por partition key
2. **C** - Cross-account Function URLs requiere resource-based policy  
3. **C** - SQS Standard puede duplicar, implementar idempotency
4. **C** - Inline > 4KB requiere S3 deployment 
5. **D** - Viewer Response para modificar respuestas a usuarios
6. **C** - Batch size grande + 3seg por evento = timeout total
7. **C** - Step Functions para workflows con wait times largos
8. **B** - Over-provisioned memory, reducir para cost optimization
9. **C** - Cache en código fuera del handler para reuse
10. **B** - 5GB files + 15min limit = dividir trabajo
11. **B** - API Gateway 29seg limit requiere async pattern
12. **B** - CodeGuru identifica bottleneck específico = optimizar eso