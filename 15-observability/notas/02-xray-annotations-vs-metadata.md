# X-Ray — Annotations vs Metadata vs Sampling

## Annotations vs Metadata

|                          | Annotations                          | Metadata                  |
| ------------------------ | ------------------------------------ | ------------------------- |
| **Indexado / filtrable** | ✅ Sí                                | ❌ No                     |
| Tipos                    | String, Number, Boolean              | Cualquier JSON            |
| Max por trace            | 50                                   | Sin límite estricto       |
| Uso                      | **Filtrar/buscar** trazas en consola | Info adicional para debug |

## Regla rápida

- **Annotation** = clave-valor indexada → **filtrás** por ella.
- **Metadata** = blob JSON → **leés** cuando ya encontraste la trace.

## Disparadores

| Pregunta dice                                                      | Concepto           |
| ------------------------------------------------------------------ | ------------------ |
| "filtrar trazas" / "expresiones de filtro" / "buscar por atributo" | **Annotations**    |
| "info adicional" / "detalles para debugging"                       | **Metadata**       |
| "controlar la tasa de captura"                                     | **Sampling rules** |
| "reducir costo de X-Ray"                                           | **Sampling**       |

## Sampling

- Default: **1 req/s + 5% del resto** (no traza todo por costo).
- Rules custom: por servicio, método HTTP, URL pattern.
- Se configuran en la **consola de X-Ray**, no en código.

## Subsegments (faltaba — error en simulacro #2)

- **Subsegment** = bloque DENTRO del segment que captura **llamadas downstream**
  (otros servicios AWS, APIs HTTP externas, queries a DB).
- Registra: servicio llamado, latencia, status, errores de cada call.
- "capturar/rastrear llamadas a otros servicios" → **subsegment** (NO metadata).
- Metadata = guardar datos; subsegment = registrar LLAMADAS.

## Trampas

- "Agregar nuevos campos al segment document" → NO existe. El schema del segment es fijo. Solo podés agregar info via **annotations** o **metadata**.
- "capturar llamadas a servicios" ≠ metadata. Metadata es blob inerte, no captura nada.

## MAPA MENTAL

- Annotation → FILTRAR (indexada)
- Metadata → LEER (blob de debug)
- Subsegment → CAPTURAR LLAMADAS (downstream: latencia/status/error)
- Sampling → CUÁNTO capturar (costo)

## Pregunta de prueba

Mandás documentos de trazado a X-Ray con `PutTraceSegments`. ¿Qué componente
incluís en el segmento para capturar las llamadas a otros servicios de AWS?

A) annotations
B) metadata
C) subsegments
D) tracing header

<details><summary>Respuesta</summary>

**C** (subsegments): registran cada llamada downstream (servicio, latencia, errores).
Cuándo sería cada una:

- **annotations** → pares clave-valor INDEXADOS para filtrar/buscar trazas.
- **metadata** → info extra NO filtrable (blob de debug), no captura llamadas.
- **tracing header** → propaga el trace ID entre servicios, no captura las llamadas.
</details>
