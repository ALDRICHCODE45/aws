# LSI vs GSI — cuándo usar cada uno

| | LSI | GSI |
| -- | --- | --- |
| **Cuándo creás** | **Solo al crear la tabla** | **Cualquier momento** |
| Cantidad max | 5 | 20 |
| **Partition Key** | **MISMA que la tabla** | **Cualquier atributo** |
| Sort Key | Distinta, obligatoria | Opcional |
| Consistencia | Strong + Eventually | **Solo Eventually** |
| Throughput | Comparte con tabla | **Independiente** |
| Eliminar | Solo borrando la tabla | Cuando quieras |

## Regla mental

- **L**SI = **L**ocked (PK fija, momento fijo).
- **G**SI = **G**lobal (PK libre, momento libre).

## Disparadores

| Pregunta dice | Índice |
| ------------- | ------ |
| "buscar por **mismo PK** + otro orden" | **LSI** |
| "buscar por **PK distinta**" / "agrupar por otro atributo" | **GSI** |
| "no se puede modificar la tabla" / "tabla ya existe" | **GSI** (LSI solo al crear) |
| "ranking por X" donde X NO es la PK original | **GSI** con X como PK |

## Trampa común

"LSI con [atributo distinto] como PK" → **IMPOSIBLE**. LSI obliga a usar la PK de la tabla. Si la opción cambia la PK → descarte automático.

## Patrón "ranking / top N por categoría"

Para "top N de [campo] por [categoría]":
- PK del GSI = el campo de **categoría**
- SK del GSI = el campo de **ranking**
- Query con `ScanIndexForward=false` para descendente
