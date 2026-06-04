# Query vs Scan y orden inverso

- **Query** = eficiente, solo toca una partición (necesita PK, opcionalmente SK)
- **Scan** = lee TODA la tabla, lento y caro — último recurso

## Orden inverso en Query

```
ScanIndexForward = false  → descendente (nuevo → viejo)
ScanIndexForward = true   → ascendente (default)
```

Trampa: el nombre dice "Scan" pero es parámetro de **Query**.
