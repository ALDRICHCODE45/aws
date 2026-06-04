# DynamoDB — LSI (Local Secondary Index)

Sort Key alternativa para tu tabla. La PK se mantiene igual.

---

## Qué es

- Mismo Partition Key que la tabla base, **diferente Sort Key**
- Permite ordenar/consultar por otro atributo sin cambiar el diseño de la tabla
- NO es un filtro extra — **reemplaza** la SK en esa consulta

## Ejemplo

- Tabla: PK = `User_ID`, SK = `Game_ID` → query por juego
- LSI: PK = `User_ID`, SK = `Game_TS` → query por fecha

## Reglas

- Máximo **5 LSI** por tabla
- Se definen **solo al crear la tabla** — no después
- Comparte RCU/WCU con la tabla base (no tiene capacidad propia)
- Soporta **strongly consistent** y eventually consistent

## Projections

Qué atributos se copian al índice:

| Projection | Qué incluye |
|---|---|
| `KEYS_ONLY` | Solo PK + SK del índice |
| `INCLUDE` | Keys + atributos que vos elegís |
| `ALL` | Todos los atributos |

Si pedís un atributo que NO está en la projection → DynamoDB va a la tabla base a buscarlo (**fetch**, caro en RCU).
