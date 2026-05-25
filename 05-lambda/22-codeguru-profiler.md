# Lambda y CodeGuru Profiler

## ¿Qué es CodeGuru Profiler?

Herramienta de **profiling de performance** que analiza el rendimiento **interno** de tu código Lambda a nivel función/método.

### Características:
- **Performance profiling** en tiempo de ejecución
- Soporta **Java y Python**
- Se activa desde **consola AWS Lambda**
- Crea **grupo de perfiles** para tu función
- Política automática: `AmazonCodeGuruProfilerAgentAccess`

---

## CloudWatch vs X-Ray vs CodeGuru Profiler

### CloudWatch Logs & Metrics
**QUÉ hace:** Logs y métricas **básicas de Lambda**
```
✅ Duration, Memory Used, Billed Duration
✅ Error count, Invocations, Throttles
✅ Cold starts, Concurrent executions
✅ Logs de aplicación (print, console.log)
```
**CUÁNDO usar:** Debugging básico, alertas, monitoreo general

### X-Ray
**QUÉ hace:** **Tracing distribuido** entre servicios
```
✅ Trace completo cross-services
✅ Lambda → DynamoDB → S3 → API Gateway
✅ Latencias entre componentes
✅ Dependency map visual
✅ Bottlenecks de arquitectura
```
**CUÁNDO usar:** Debugging **microservicios complejos**, encontrar bottlenecks entre servicios

### CodeGuru Profiler  
**QUÉ hace:** **Performance profiling a nivel código**
```
✅ ¿Qué función/método consume más tiempo?
✅ ¿Qué líneas específicas son lentas?
✅ Memory allocation patterns
✅ CPU usage por función
✅ Flame graphs y call stacks
```
**CUÁNDO usar:** **Optimización interna** de código, performance tuning

---

## Ejemplo práctico de debugging

**Problema:** "Mi Lambda de ecommerce tarda 5 segundos"

### 1. CloudWatch te dice:
```
Duration: 5000ms (muy lento!)
Memory: 512MB usado de 1024MB
Error rate: 5%
Invocations: 1000/hour
```
**→ Confirma que HAY un problema**

### 2. X-Ray te dice:
```
Total request: 5000ms
├─ Lambda execution: 1000ms
├─ DynamoDB query: 3000ms ← BOTTLENECK EXTERNO
└─ S3 upload: 1000ms
```
**→ El problema está en DynamoDB (mal diseñado query o falta índice)**

### 3. CodeGuru te dice:
```
Lambda internal (1000ms):
├─ processOrder(): 800ms
│   ├─ validatePayment(): 600ms ← CÓDIGO LENTO
│   ├─ calculateTax(): 150ms  
│   └─ saveOrder(): 50ms
└─ Other: 200ms
```
**→ validatePayment() está mal implementado (loop ineficiente, llamadas sync innecesarias)**

---

## Configuración

### Activación automática:
1. **Consola Lambda** → Configuration → Monitoring tools
2. **CodeGuru Profiler** → Enable
3. AWS añade automáticamente:
   - Capa CodeGuru Profiler
   - Variables de entorno
   - Política `AmazonCodeGuruProfilerAgentAccess`

### Variables de entorno añadidas:
```
AWS_CODEGURU_PROFILER_GROUP_NAME=aws-lambda-MyFunction
AWS_CODEGURU_PROFILER_ENABLED=true
```

---

## Casos de uso para examen

### ✅ Usar CodeGuru Profiler cuando:
- **Optimizar performance** de función específica
- **Identificar código lento** dentro de la función
- **Memory leaks** o allocation issues
- **CPU hotspots** en el código
- **Función compleja** con múltiples métodos internos

### ❌ NO usar cuando:
- Problema está **entre servicios** → usar **X-Ray**
- Solo necesitás **logs básicos** → usar **CloudWatch**
- **Debugging de errores** → usar **CloudWatch Logs**
- **Monitoreo de infraestructura** → usar **CloudWatch Metrics**

---

## Para el examen

**Pregunta típica:** "Optimizar performance interna de función Lambda específica"
**Respuesta:** CodeGuru Profiler

**Diferenciación clave:**
- **CloudWatch** = ¿hay problema? (métricas generales)
- **X-Ray** = ¿dónde está el problema? (entre servicios) 
- **CodeGuru** = ¿qué línea específica es el problema? (dentro del código)

**Key facts:**
- Solo **Java y Python**
- **Profiling en tiempo real** de ejecución
- **Flame graphs** para visualizar performance
- **Activación simple** desde consola Lambda
- **Complementa** CloudWatch y X-Ray (no los reemplaza)