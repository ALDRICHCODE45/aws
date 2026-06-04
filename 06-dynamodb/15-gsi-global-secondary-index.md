# DynamoDB — GSI (Global Secondary Index)

Clave primaria completamente diferente a la tabla base. Como tener una segunda tabla con los mismos datos reorganizados.

---

## Qué es

- PK **diferente** a la tabla base (y SK opcional, también diferente)
- Permite hacer Query por atributos que en la tabla base no son PK

## Ejemplo

- Tabla: PK = `User_ID`, SK = `Game_ID` → "dame los juegos del usuario X"
- GSI: PK = `Game_ID`, SK = `Game_TS` → "dame los usuarios que jugaron el juego Y"
- Sin el GSI esa segunda consulta requeriría un Scan (caro)

## Reglas

- Máximo **20 GSI** por tabla
- Se pueden crear/modificar **en cualquier momento**
- Tiene **sus propias RCU/WCU** (separadas de la tabla)
- **Solo eventually consistent** — no soporta strongly consistent
- Mismas projections que LSI: `KEYS_ONLY`, `INCLUDE`, `ALL`

---

## LSI vs GSI — Tabla comparativa

| | LSI | GSI |
|---|---|---|
| PK | Misma que la tabla | Diferente |
| SK | Alternativa | Alternativa (opcional) |
| Cuándo crear | Solo al crear la tabla | En cualquier momento |
| Capacidad | Comparte con la tabla | Propias RCU/WCU |
| Límite | 5 | 20 |
| Consistencia | Strongly o Eventually | **Solo Eventually** |

## GSI arrastra a la tabla base (throttling)

Cada escritura en la tabla se replica automáticamente al GSI. Si el GSI no tiene suficientes WCU:

1. Escritura en tabla base → OK
2. Réplica al GSI → throttling (pocas WCU)
3. DynamoDB **frena la tabla base** para no perder sincronización

→ La tabla puede tener WCU de sobra y aun así sufrir throttling por culpa de un GSI subaprovisionado.

**Con LSI no pasa** porque usa las mismas WCU de la tabla.

## Trampas de examen

- "Necesito strongly consistent en un índice secundario" → **LSI**, no GSI
- "Necesito consultar por un atributo que no es la PK" → **GSI**
- "La tabla tiene throttling pero las WCU están bien" → revisar GSI subaprovisionado
- "Elegí bien la PK del GSI" → si el GSI tiene hot partition, arrastra a la tabla entera
