# DynamoDB — Batch operations (reducir round-trips)

## Problema típico
App lee 1 item → modifica → escribe 1 item → repite N veces (secuencial) → LENTO.
"Reducir tiempo con configuración mínima" → **BatchGetItem + BatchWriteItem**.

## Por qué
- Secuencial = N×2 viajes de red (cada GetItem/PutItem tiene latencia). ESE es el cuello.
- Batch agrupa muchas operaciones en POCAS llamadas → menos round-trips → más rápido.
- "Config mínima" = solo cambiás las llamadas en el código, sin infra nueva.

## Límites (memorizar)
- **BatchGetItem**: hasta **100 items** o 16 MB por llamada.
- **BatchWriteItem**: hasta **25 items** (Put/Delete) o 16 MB. Item máx 400 KB.
  - NO soporta UpdateItem. NO soporta conditional writes.

## Mito a romper (error propio)
"Batch es solo si los items llegan TODOS JUNTOS; si vienen de a uno no puedo."
FALSO. El "de a uno" es la forma ineficiente ACTUAL, no una restricción.
Si tenés un conjunto conocido (ej: 100 entradas), vos los AGRUPÁS en batches.
Batch = agrupar lo que YA tenés en menos requests.

## Distractores
- ❌ Multithreading → más complejidad (no "config mínima"); siguen siendo N×2
  llamadas individuales → throttling. Trampa de "meté código".
- ❌ Cluster EC2 → tirar hardware = más infra = lo opuesto a config mínima.
- ❌ Conditional Writes → control de concurrencia/evitar sobrescritura, NO velocidad.
  Distractor de tema equivocado.

## Gancho
"muchas ops de un solo item, lentas + config mínima" → Batch{Get,Write}Item.

## Pregunta de prueba

Una app lee y escribe 100 ítems de a uno (secuencial) y tarda demasiado.
¿Qué estrategia reduce el tiempo con configuración mínima?

A) Modificar la app para usar multithreading
B) Usar `BatchGetItem` y `BatchWriteItem`
C) Desplegar la app en un clúster de EC2
D) Usar escrituras condicionales (Conditional Writes)

<details><summary>Respuesta</summary>

**B**: batch agrupa muchas ops en pocas llamadas → menos round-trips, sin infra nueva.
Cuándo sería cada una:
- **multithreading** → más complejidad (no "config mínima"), siguen N×2 llamadas.
- **clúster EC2** → tirar hardware = más infra, lo opuesto a mínimo.
- **conditional writes** → control de concurrencia, no velocidad.
</details>
