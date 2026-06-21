# DynamoDB — Atomic counters vs Conditional writes vs Streams

## Distinción clave
| Necesidad | Solución |
| --------- | -------- |
| Contador donde **aproximado está OK** + config mínima | **Atomic counter** (`ADD` / `SET x = x + 1` en UpdateItem) |
| Valor que necesita **exactitud / sin lost updates** | **Conditional writes** / optimistic locking (version) |
| Reaccionar a **cambios** de la tabla (trigger, replicación) | **DynamoDB Streams** |

## Atomic counter
- `UpdateItem` con `ADD` → incrementa atómicamente SIN leer primero. 1 llamada, cero infra.
- NO idempotente: un reintento puede contar de más.
- Caso de uso: vistas, likes, visitantes → "aproximado está bien".

## Pregunta tipo (visitantes, aproximado)
"contar visitantes, idea aproximada, se tolera sobre/subconteo, config mínima"
→ **Atomic counter**. La tolerancia al error ES la pista: NO necesitás conditional writes.

## Distractores
- ❌ **DynamoDB Streams** → NO cuenta; es change-log. Para contar necesitarías
  Lambda consumiendo el stream = config extra. Lo opuesto a mínima.
- ❌ **Conditional writes** ("solo si nuevo > actual") → para exactitud estricta;
  acá sobra. Encima descarta incrementos concurrentes (semántica equivocada).
- ❌ **ReturnConsumedCapacity=TOTAL** → solo metadata de capacidad. Ruido.

## Patrón propio (#4)
Buscar "más preciso/robusto" cuando el enunciado dice que el error se tolera =
sobre-ingeniería. Leer QUÉ optimizan: aquí era SIMPLICIDAD, no exactitud.

## Pregunta de prueba

Querés contar visitantes de un sitio (idea aproximada, se tolera pequeño sobre/subconteo)
con configuración mínima en DynamoDB. ¿Qué usás?

A) Habilitar DynamoDB Streams para seguir a los visitantes
B) Atomic counters (`UpdateItem` con `ADD`) por cada visitante
C) Conditional writes (solo si el nuevo valor es mayor)
D) Escrituras con `ReturnConsumedCapacity = TOTAL`

<details><summary>Respuesta</summary>

**B** (atomic counter): 1 llamada, sin infra; aproximado es suficiente.
Cuándo sería cada una:
- **Streams** → reaccionar a CAMBIOS (trigger), no contar; requiere consumidor = config extra.
- **conditional writes** → cuando necesitás exactitud estricta (acá sobra).
- **ReturnConsumedCapacity** → solo metadata de capacidad, no cuenta nada.
</details>
