# X-Ray — Annotations vs Metadata vs Sampling

## Annotations vs Metadata

| | Annotations | Metadata |
| -- | ----------- | -------- |
| **Indexado / filtrable** | ✅ Sí | ❌ No |
| Tipos | String, Number, Boolean | Cualquier JSON |
| Max por trace | 50 | Sin límite estricto |
| Uso | **Filtrar/buscar** trazas en consola | Info adicional para debug |

## Regla rápida

- **Annotation** = clave-valor indexada → **filtrás** por ella.
- **Metadata** = blob JSON → **leés** cuando ya encontraste la trace.

## Disparadores

| Pregunta dice | Concepto |
| ------------- | -------- |
| "filtrar trazas" / "expresiones de filtro" / "buscar por atributo" | **Annotations** |
| "info adicional" / "detalles para debugging" | **Metadata** |
| "controlar la tasa de captura" | **Sampling rules** |
| "reducir costo de X-Ray" | **Sampling** |

## Sampling

- Default: **1 req/s + 5% del resto** (no traza todo por costo).
- Rules custom: por servicio, método HTTP, URL pattern.
- Se configuran en la **consola de X-Ray**, no en código.

## Trampa

"Agregar nuevos campos al segment document" → NO existe. El schema del segment es fijo. Solo podés agregar info via **annotations** o **metadata**.
