# DynamoDB — Atomic counters vs Conditional writes vs Streams

## Distinción clave

| Necesidad                                                   | Solución                                                   |
| ----------------------------------------------------------- | ---------------------------------------------------------- |
| Contador donde **aproximado está OK** + config mínima       | **Atomic counter** (`ADD` / `SET x = x + 1` en UpdateItem) |
| Valor que necesita **exactitud / sin lost updates**         | **Conditional writes** / optimistic locking (version)      |
| Reaccionar a **cambios** de la tabla (trigger, replicación) | **DynamoDB Streams**                                       |

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

## Conditional writes / Optimistic locking (el sentido CONTRARIO)

- Disparador textual: **"evitar sobrescribir un valor si cambió desde la última lectura"**,
  "evitar lost updates", precio/stock/estado con actualizaciones concurrentes.
- Cómo: leés el ítem (con atributo `version` o el valor actual) → escribís con
  **`ConditionExpression`** ("actualizá solo si version/valor == el que leí").
  Si otro lo cambió → la escritura FALLA → re-leés y reintentás.
- **Atomic counter NO sirve acá**: incrementa a ciegas sin condicionar al valor previo (semántica opuesta).

## Los DOS sentidos (no caer en el surco de uno solo)

| Pista en el enunciado | Respuesta |
| --------------------- | --------- |
| "aproximado OK", "se tolera sobre/subconteo", likes/vistas/visitantes | **Atomic counter** (`ADD`) |
| "no sobrescribir si cambió desde la última lectura", "evitar lost updates", precio/stock | **Conditional writes / optimistic locking** (version) |

## Patrón propio (#4)

Buscar "más preciso/robusto" cuando el enunciado dice que el error se tolera =
sobre-ingeniería. Leer QUÉ optimizan: aquí era SIMPLICIDAD, no exactitud.

## Patrón propio (#5) — ERROR RECURRENTE

Ya fallé esto ANTES en el mismo sentido: vi "concurrencia en DynamoDB" y disparé
"atomic counter" por reflejo. Entreno los DOS sentidos. La frase "si cambió desde
la última lectura" = SIEMPRE conditional writes / optimistic locking, NUNCA atomic counter.

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

## Pregunta de prueba ESPEJO (sentido opuesto)

App de reservas: múltiples usuarios actualizan disponibilidad/precio de boletos a la vez.
La lógica debe **evitar que un proceso sobrescriba un valor si el precio cambió desde la
última lectura**. ¿Qué solución asegura integridad en las actualizaciones concurrentes?

A) Bloqueo pesimista usando una tabla auxiliar como control
B) Bloqueo optimista mediante escrituras condicionales
C) Contadores atómicos para el control de concurrencia
D) Operaciones `BatchWriteItem` para múltiples actualizaciones

<details><summary>Respuesta</summary>

**B**: optimistic locking con conditional writes (version + `ConditionExpression`).
La frase "no sobrescribir si cambió desde la última lectura" = optimistic locking SIEMPRE.

- **A** → pesimista con tabla auxiliar = sobre-ingeniería, mata concurrencia; no idiomático en DynamoDB.
- **C** → atomic counter incrementa a ciegas sin condicionar al valor previo (semántica opuesta). TRAMPA recurrente.
- **D** → `BatchWriteItem` es para throughput; ni soporta `ConditionExpression`, no maneja conflictos.
</details>
