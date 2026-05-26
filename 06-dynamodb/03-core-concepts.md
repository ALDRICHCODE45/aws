# DynamoDB — Conceptos Básicos

## Vocabulario SQL → DynamoDB

| SQL relacional | DynamoDB              |
|----------------|-----------------------|
| Tabla          | Tabla                 |
| Fila / Row     | **Item** (elemento)   |
| Columna        | **Attribute** (atributo) |
| Celda          | Valor del atributo    |

## Diferencia clave con SQL

En SQL, **todas las filas tienen las mismas columnas** (algunas en NULL).
En DynamoDB, **cada item puede tener atributos diferentes**:

```
Item 1: { user_id, nombre, edad, email }
Item 2: { user_id, nombre, telefono, direccion }
```

Lo **único obligatorio** en todos los items es la **Primary Key**. El resto es libre y puede agregarse con el tiempo.

## Componentes de una tabla

- Cada tabla tiene una **Primary Key** (se define al crear la tabla, no se puede cambiar después).
- Cada tabla puede tener un número **infinito de items**.
- Cada item tiene atributos (pueden agregarse con el tiempo, pueden ser nulos).
- El **tamaño máximo de un item es 400 KB**.

## Por qué el límite de 400 KB tiene sentido

NO es un capricho. Es una decisión de diseño deliberada:

1. **Latencia predecible** — items grandes romperían el SLA de "single-digit millisecond".
2. **Costo controlado** — lecturas se cobran por bloques de 4 KB, escrituras por bloques de 1 KB. Items gigantes = costos gigantes.
3. **Replicación eficiente** — DynamoDB replica en 3 AZ. Items grandes saturarían la red.

### Solución correcta para datos grandes (PREGUNTA DE EXAMEN)

Patrón **Large Object Storage**:

- Datos grandes (imágenes, PDFs, videos) → guardar en **S3**.
- En DynamoDB guardar solo la **referencia (URL o key de S3)**.

## Tipos de datos

DynamoDB agrupa los tipos en 3 categorías según su forma:

### 1. Escalares — "un valor simple, atómico"

| Tipo    | Ejemplo                |
|---------|------------------------|
| String  | `"Aldrich"`            |
| Number  | `42`, `3.14`           |
| Binary  | bytes crudos (base64)  |
| Boolean | `true` / `false`       |
| Null    | ausencia explícita     |

> Mnemotécnico: **"cosas que entran en una sola celda de Excel"**.

### 2. Documento — "una estructura anidada"

| Tipo | Ejemplo                                  |
|------|------------------------------------------|
| List | `["hola", 42, true]` (tipos mixtos, ordenado) |
| Map  | `{ "nombre": "Aldrich", "edad": 30 }`    |

> Mnemotécnico: **"como JSON"**. Si lo escribís en JSON, es tipo documento.

### 3. Conjuntos (Sets) — "colección sin duplicados, sin orden, de UN solo tipo"

| Tipo        | Ejemplo                |
|-------------|------------------------|
| String Set  | `["rojo", "verde", "azul"]` |
| Number Set  | `[1, 2, 3]`            |
| Binary Set  | bytes únicos           |

> Mnemotécnico: **"como un Set matemático"**. Sin duplicados, sin orden, todos del mismo tipo.

## List vs Set (cae en examen)

| List                          | Set                          |
|-------------------------------|------------------------------|
| Permite duplicados            | NO permite duplicados        |
| Mantiene orden                | NO mantiene orden            |
| Tipos mixtos                  | Un solo tipo                 |
| Acceso por índice `lista[0]`  | Solo operaciones de conjunto |
