# ProjectionExpression vs FilterExpression

Me pegó DOS veces en el examen. No confundir.

- **ProjectionExpression** → QUÉ ATRIBUTOS traer (como SELECT col1, col2)
- **FilterExpression** → QUÉ ÍTEMS devolver (como WHERE, pero NO reduce RCU)

> "Seleccionar solo determinados atributos" → SIEMPRE `ProjectionExpression`
