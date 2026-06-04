# DynamoDB — Tipos de escritura

---

| Tipo | Qué pasa | Resultado |
|---|---|---|
| **Concurrentes** | Dos escriben un valor fijo al mismo ítem | La segunda sobreescribe a la primera (last writer wins) |
| **Atómicas** | Dos incrementan el valor (+1 y +2) | Ambas pasan, el valor sube 3. No se pisan. |
| **Condicionales** | Dos escriben "solo si valor = 0" | La primera pasa, la segunda falla (`ConditionalCheckFailedException`) |
| **Por lotes** | Escribir/actualizar muchos ítems a la vez | `BatchWriteItem` — múltiples operaciones en una llamada |
