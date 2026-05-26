# DynamoDB — Primary Keys (Claves Primarias)

DynamoDB ofrece **2 opciones de Primary Key**. La elección se hace al crear la tabla y **no se puede cambiar después**.

---

## Opción 1: Partition Key (HASH)

Solo una clave que identifica el item.

### Reglas

- Debe ser **única** para cada item.
- Debe ser **diversa** (alta cardinalidad) para distribuir bien los datos.

### Por qué se llama HASH

DynamoDB es una base distribuida con muchas particiones físicas. Internamente hace:

```
hash(partition_key) → decide en qué partición física vive el item
```

La partition key literalmente decide **en qué pedazo físico de almacenamiento** va el item.

### Por qué tiene que ser diversa — Hot Partition

Si usás una partition key con poca diversidad (ej: `pais` y el 90% son `AR`):

- Todos esos items caen en la misma partición.
- Esa partición se satura → **hot partition**.
- Latencia explota, AWS te tira throttling.

### Buenos vs malos ejemplos

| Buena Partition Key | Mala Partition Key |
|---------------------|--------------------|
| `user_id` (UUID)    | `status` (solo 3 valores) |
| `order_id`          | `country`                 |
| `device_id`         | `tipo_cliente`            |

> **Regla mental:** Partition Key = "¿dónde guardo este item?" → respuesta bien repartida.

---

## Opción 2: Partition Key + Sort Key (HASH + RANGE)

Dos claves combinadas. La PK sola **puede repetirse**; lo único es la **combinación**.

### Ejemplo

```
User_ID         Game_ID    Score   Result
7791a3d6...     4421       92      Win
873e0634...     1894       14      Lose
873e0634...     4521       77      Win    ← User_ID repetido, OK
```

Acá `873e0634` aparece dos veces porque la combinación con `Game_ID` es distinta.

### Qué pasa físicamente

1. Todos los items con la **misma Partition Key** se guardan **juntos en la misma partición**.
2. Dentro de esa partición, los items están **ordenados por Sort Key**.

Por eso se llama "Sort Key" / "Range Key" — literalmente ordena los items dentro de la partición.

### Por qué es poderoso

Habilita queries eficientes del tipo:

- "Todos los juegos del usuario X" → rápido, están todos juntos.
- "Juegos del usuario X con Game_ID entre 1000 y 5000" → rápido, están ordenados.
- "Últimos 10 juegos del usuario X" → rápido, sort key da el orden.

Sin sort key, esto requeriría un `Scan` (lento y caro) en vez de un `Query` (barato).

---

## Analogía — Edificio de departamentos

- **Partition Key** = número de edificio (a qué edificio entrar)
- **Sort Key** = número de departamento dentro del edificio (qué puerta tocar)

- **Opción 1** (solo Partition): cada persona vive en su propio edificio único.
- **Opción 2** (Partition + Sort): varias personas comparten edificio, pero cada una tiene su departamento. La dirección completa (edificio + depto) es lo único.

Y los departamentos están **numerados en orden** → preguntar "edificio 5, deptos 1 al 10" es rapidísimo.

---

## Cuándo usar cada una

| Caso                                              | Opción                |
|---------------------------------------------------|-----------------------|
| Items independientes sin relaciones               | Solo Partition Key    |
| Agrupar items relacionados y consultarlos juntos  | Partition + Sort      |
| Tabla de usuarios                                 | `user_id`             |
| Pedidos de un usuario                             | `user_id` + `order_id` |
| Mensajes de un chat ordenados por fecha           | `chat_id` + `timestamp` |
| Posts de un usuario con paginación                | `user_id` + `post_timestamp` |

> **Pista del examen:** si dice "ordenados por X" o "todos los X de un Y" → casi seguro Partition + Sort.

---

## Trampas típicas del examen

1. **"¿Cuál sería una buena partition key?"** → única y diversa (alta cardinalidad).
2. **"Hot partition"** → la respuesta correcta menciona partition key con poca diversidad.
3. **"Cómo consulto items relacionados eficientemente?"** → Partition + Sort, con `Query` (no `Scan`).
4. **"La partition key puede repetirse?"** → solo si hay sort key; lo único es la combinación.
