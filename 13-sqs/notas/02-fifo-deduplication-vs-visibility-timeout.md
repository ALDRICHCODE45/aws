# SQS FIFO — Deduplicación (NO confundir con VisibilityTimeout)

## El problema típico

Cola FIFO + el productor manda mensajes DUPLICADOS → "¿cómo reduzco procesar duplicados?"

## Respuesta correcta

**`MessageDeduplicationId`** en la llamada `SendMessage`.

- Dos mensajes con el mismo dedup ID dentro de la **ventana de 5 min** → el 2º
  se descarta, no se entrega.
- Variante automática: **`ContentBasedDeduplication`** → genera el dedup ID
  desde un hash SHA-256 del body.

## Distractores y por qué son trampa

- ❌ "VisibilityTimeout descarta duplicados" → **NO existe esa capacidad**.
  VisibilityTimeout solo OCULTA un mensaje mientras se procesa. No deduplica.
- ❌ "Usar cola Standard en vez de FIFO" → AL REVÉS. Standard = at-least-once =
  MÁS duplicados.
- ❌ "Refactorizar Lambda para descartar contenido similar" → dedup casero;
  funciona pero el examen premia la solución NATIVA (MessageDeduplicationId).

## Reglas anti-trampa (de error propio)

1. Si una opción describe una feature que NO EXISTE pero suena bien → distractor
   (igual que `s3:PostObject`, `azBalanced`).
2. NO anclarse en el concepto recién estudiado (VisibilityTimeout estaba fresco
   y por eso cayó). Leer TODAS las opciones.
3. Solución nativa/gestionada > solución casera en la app.

## No mezclar

- **MessageDeduplicationId** = evitar que el MISMO mensaje entre dos veces (FIFO).
- **MessageGroupId** = orden dentro de un grupo (FIFO).
- **VisibilityTimeout** = evitar que OTRO consumer agarre un mensaje en proceso
  (cualquier cola). Corto de más → reprocesa.

## Pregunta de prueba

Una cola FIFO de SQS recibe mensajes duplicados intermitentes del productor.
¿Qué acción reduce el procesamiento de duplicados?

A) Configurar la cola para descartar duplicados durante el VisibilityTimeout
B) Agregar `MessageDeduplicationId` en la llamada `SendMessage`
C) Cambiar a una cola Standard
D) Usar `MessageGroupId` para los mensajes

<details><summary>Respuesta</summary>

**B** (`MessageDeduplicationId`): dedup nativo de FIFO en ventana de 5 min.
Cuándo sería cada una:

- **VisibilityTimeout descarta duplicados** → no existe esa capacidad (trampa).
- **Standard** → al revés, genera MÁS duplicados (at-least-once).
- **MessageGroupId** → es para ORDEN dentro de un grupo, no para deduplicar.
</details>
