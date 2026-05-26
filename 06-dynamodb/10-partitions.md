# DynamoDB — Particiones Internas

Las particiones son la base física de cómo DynamoDB escala. Entender esto es **crítico** para diseñar bien y evitar problemas en producción.

---

## ¿Por qué existen las particiones?

DynamoDB promete:
- Millones de requests/segundo
- Cientos de TB de datos
- Latencia de milisegundos

**Ningún servidor del mundo aguanta eso solo.** La única forma física de cumplir esa promesa es **partir los datos entre muchos servidores**. Cada uno de esos pedazos es una **partición**.

> **Partición = un servidor físico (con su disco, CPU, RAM) que guarda un pedazo de tu tabla.**

Las particiones no son un capricho — son la única forma de escalar a esos números.

---

## Analogía — biblioteca gigante

Imaginá una biblioteca con **10 millones de libros**.

### Opción mala — 1 sola estantería

- Una persona buscando un libro → OK.
- 1000 personas al mismo tiempo → se amontonan, caos.
- La estantería se cae por el peso.

Esto sería DynamoDB sin particiones. Imposible.

### Opción buena — 1000 estanterías repartidas

- Cada estantería tiene 10.000 libros.
- Cada estantería tiene su propio bibliotecario.
- 1000 personas buscando → cada una va a una estantería distinta, no se traban.

**Cada estantería = una partición.**
**Cada bibliotecario = la capacidad (RCU/WCU) de esa partición.**

---

## ¿Cómo decide DynamoDB en qué partición va cada item?

Cuando insertás un item:

```
1. Toma la Partition Key del item       → ej: "USER#u1"
2. La pasa por una función hash         → ej: hash("USER#u1") = 7f3a9b...
3. Ese hash determina la partición      → "este item va a la partición 5"
```

**Por eso se llama "Partition Key" — literalmente decide la partición.**
Y por eso en `04-primary-keys.md` se la llamaba **HASH key**: porque ANTES de llegar al storage, pasa por un hash.

---

## El diagrama mental

```
        ┌─────────────────────────┐
        │   Función Hash          │
        │   (toma Partition Key)  │
        └────────┬────────────────┘
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
   ┌───────┐ ┌───────┐ ┌───────┐
   │ Part1 │ │ Part2 │ │ Part3 │
   │ items │ │ items │ │ items │
   └───────┘ └───────┘ └───────┘
```

Los datos entran por arriba, pasan por el hash, y el hash decide a qué partición van. Cada partición es independiente, con sus propios items.

---

## RCU/WCU se reparten UNIFORMEMENTE entre particiones

Esto es CRÍTICO. Si provisionás **1000 RCU** en una tabla con **10 particiones**, NO tenés 1000 RCU "a libre demanda".

> Tenés **100 RCU por cada partición** (1000 ÷ 10).

---

## El problema clásico: Hot Partition

Imaginá esta tabla:

```
Tabla USERS (10 particiones, 1000 RCU totales, 100 RCU por partición)

Partition Key      cae en partición
USER#u1            Partición 3
USER#u2            Partición 7
USER#famoso        Partición 5   ← ¡atención!
```

`USER#famoso` resulta ser un usuario popular y recibe **5000 requests/seg**:

- Esa partición solo tiene **100 RCU**.
- Recibe ~5000 RCU de demanda.
- Resultado → `ProvisionedThroughputExceededException` (throttling).

**Y lo peor:** las otras 9 particiones están **idle**, pero NO pueden "prestar" su capacidad. Cada partición tiene su capacidad propia y aislada.

Esto es **Hot Partition** — uno de los problemas más comunes en DynamoDB mal diseñado.

---

## Casos de uso reales — cuándo importa

### Caso 1 — Red social

Si usás `country_code` como Partition Key:
- 90% de usuarios son de Argentina → todos a la misma partición.
- Solución: usar `user_id` (UUID) → reparto uniforme.

### Caso 2 — Logs por día

Si usás `date` (ej: `2026-05-25`) como Partition Key:
- TODOS los logs del día caen en la misma partición.
- Solución: agregar sufijo random → `date#shard_random` (write sharding).

### Caso 3 — IoT con sensores desiguales

Si un sensor envía 10× más datos que los otros y usás `sensor_id` como PK:
- Esa partición se satura.
- Solución: write sharding → `sensor_id#0`, `sensor_id#1`, ..., `sensor_id#9`.

---

## Regla de oro

> **Una buena Partition Key reparte los datos Y el tráfico de forma UNIFORME entre todas las particiones.**

Tiene que cumplir DOS cosas:

1. **Alta cardinalidad** (muchos valores únicos posibles).
2. **Acceso uniforme** (no que el 90% del tráfico vaya al mismo valor).

| Buena Partition Key | Mala Partition Key                    |
|---------------------|---------------------------------------|
| `user_id` (UUID)    | `status` (solo 3 valores)             |
| `order_id`          | `country` (sesgado por Argentina)     |
| `device_id`         | `date` (todos consultan el de hoy)    |

---

## Trampas típicas del examen

1. **"Hot partition"** → la respuesta menciona Partition Key con poca diversidad o tráfico desigual.

2. **"La app recibe throttling pero el monitoreo muestra capacidad total no agotada"** → es hot partition: una partición se saturó, las demás están idle.

3. **"Cómo distribuir mejor el tráfico"** → rediseñar la PK, o **write sharding** (sufijo random).

4. **"Cómo evitar hot partition con datos por fecha"** → write sharding (`date#shard_random`).

5. **AWS calcula la cantidad de particiones automáticamente** según capacidad y tamaño. No lo controlás manualmente.

---

## Auto-test mental

1. ¿Por qué DynamoDB usa particiones? Una razón técnica.
2. ¿Qué algoritmo decide a qué partición va un item?
3. Si tenés 500 RCU provisionadas y 5 particiones, ¿cuántos RCU tiene cada partición?
4. ¿Qué pasa si una sola partición recibe más tráfico que su capacidad, aunque las otras estén idle?
5. ¿Cuáles son las DOS características de una buena Partition Key?
6. Si querés guardar logs diarios sin hot partition, ¿qué estrategia usás?

### Respuestas

1. Ningún servidor solo aguanta millones de req/s + cientos de TB. Hay que partir físicamente los datos.
2. Una función hash sobre la Partition Key.
3. 100 RCU por partición (500/5).
4. Throttling (`ProvisionedThroughputExceededException`), aunque la capacidad total no se haya agotado. **Hot partition**.
5. Alta cardinalidad + acceso uniforme.
6. Write sharding: agregar un sufijo random a la Partition Key (`date#0`, `date#1`, ...).
