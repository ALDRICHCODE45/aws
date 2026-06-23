# DynamoDB — ReturnConsumedCapacity (TOTAL vs INDEXES) y ReturnValues

Dos ejes: qué PARÁMETRO + qué VALOR. No confundir capacidad con atributos del ítem.

## Eje 1 — qué parámetro
- **`ReturnConsumedCapacity`** → info de **capacidad consumida** (CU).
- **`ReturnValues`** → devuelve **atributos del ÍTEM** antes/después de la escritura
  (`NONE`, `ALL_OLD`, `UPDATED_OLD`, `ALL_NEW`, `UPDATED_NEW`). NADA de capacidad.
  Ojo: `TOTAL`/`INDEXES` NO son valores válidos de `ReturnValues` (distractor inválido).

## Eje 2 — valor de ReturnConsumedCapacity
| Valor       | Devuelve                                                       |
| ----------- | -------------------------------------------------------------- |
| `NONE`      | nada (default)                                                 |
| `TOTAL`     | **solo el total agregado** consumido                          |
| `INDEXES`   | **el total + el desglose** por tabla y por cada índice afectado |

## Trampa (caí acá)
- El enunciado dice "tanto el **total** como el desglose" → la palabra "total" tira
  magnéticamente al valor `TOTAL`. PERO `INDEXES` **ya incluye el total** + el desglose.
- La señal real es **"desglose / por índice / breakdown"** → `INDEXES`, siempre.

## Ganchos
"desglose por tabla e índices" → `ReturnConsumedCapacity = INDEXES` (incluye el total).
`ReturnValues` = atributos del ítem, no capacidad. La palabra "total" es ruido si piden desglose.

## Pregunta de prueba

En cada escritura a DynamoDB necesitás que se devuelva el consumo de capacidad: el total
**y** el desglose entre la tabla y los índices secundarios afectados. ¿Qué configurás?

A) `ReturnConsumedCapacity = TOTAL`
B) `ReturnValues = INDEXES`
C) `ReturnValues = TOTAL`
D) `ReturnConsumedCapacity = INDEXES`

<details><summary>Respuesta</summary>

**D** (`ReturnConsumedCapacity = INDEXES`): da el total + desglose por tabla e índices.
- **A** `TOTAL` → solo el agregado, sin desglose (trampa de la palabra "total").
- **B / C** `ReturnValues` → es para atributos del ítem, no capacidad; `INDEXES`/`TOTAL` no son valores válidos.
</details>
