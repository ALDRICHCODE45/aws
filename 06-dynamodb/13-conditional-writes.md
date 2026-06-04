# DynamoDB — Conditional Writes (Escrituras Condicionales)

Permiten agregar condiciones a operaciones de escritura: **PutItem**, **UpdateItem**, **DeleteItem**, **BatchWriteItem**.

---

## Expresiones de condición disponibles

- `attribute_exists` / `attribute_not_exists`
- `attribute_type`
- `contains` (para strings)
- `begins_with` (para strings)
- `IN` (lista de valores)
- `BETWEEN` (rango)
- `size` (longitud del string)
- Se pueden combinar con AND, OR, NOT

---

## Filter Expression vs Condition Expression

| | Filter Expression | Condition Expression |
|---|---|---|
| Operación | Lecturas (Query/Scan) | Escrituras (Put/Update/Delete) |
| Cuándo evalúa | DESPUÉS de leer — ya consumió RCU | ANTES de escribir — si falla, NO consume WCU |
| Si no hay match | Pagás las RCU igual | No pagás WCU, devuelve `ConditionalCheckFailedException` |

---

## Trampas de examen

1. **"Reducir RCU con Filter Expression"** → NO las reduce. Filtra después de leer, ya pagaste.
2. **"Conditional Write falló, ¿consumió WCU?"** → NO. Si la condición falla, no se ejecuta la escritura.
3. Para reducir RCU en lecturas → mejor PK/SK o un índice (GSI/LSI), NO filter expressions.
