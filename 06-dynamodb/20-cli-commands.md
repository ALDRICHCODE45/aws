# DynamoDB — CLI Commands (Conviene saber)

---

## Expressions

- `--projection-expression` → qué atributos traer (como SELECT col1, col2)
- `--filter-expression` → filtra resultados antes de devolvértelos (recuerda: NO reduce RCU)

## Paginación

| Flag | Qué hace |
|---|---|
| `--page-size` | Pide de a páginas chicas por detrás, pero te muestra TODO al final. Útil si hay timeouts. |
| `--max-items` | Limita cuántos ítems te muestra. Devuelve `NextToken`. |
| `--starting-token` | Usa el `NextToken` para pedir la siguiente página. |

## Trampa de examen

- "Timeout al listar tabla grande" → `--page-size` (páginas más chicas)
- "Ver solo los primeros N resultados" → `--max-items`
- `--page-size` NO limita lo que ves, solo CÓMO lo pide
