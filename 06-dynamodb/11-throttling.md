# DynamoDB — Throttling (Estrangulamiento)

Cuando superás la capacidad provisionada, DynamoDB rechaza requests con `ProvisionedThroughputExceededException`. Acá vemos **por qué pasa** y **cómo resolverlo**.

---

## El error: `ProvisionedThroughputExceededException`

Aparece cuando:
1. Superás los RCU o WCU provisionados, **Y**
2. La Burst Capacity ya se gastó (ver `06-capacity-modes.md`).

Pero la causa raíz puede ser de **3 tipos distintos**.

---

## Las 3 causas de throttling

### 1. Hot Key (Clave caliente)

**UN solo item específico** (una Partition Key) recibe demasiado tráfico.

Ejemplo: el item `PRODUCT#iphone-15` recibe 10.000 lecturas/seg porque acaba de salir a la venta, mientras los otros 999 items casi no se tocan.

### 2. Hot Partition (Partición caliente)

La **partición entera** está saturada. Puede ser por muchos items distintos accedidos al mismo tiempo dentro de esa partición.

Ejemplo: usaste `country_code` como PK, 90% de tus usuarios son de Argentina → una partición concentra el 90% del tráfico.

### 3. Elementos muy grandes

Recordá: RCU/WCU dependen del **tamaño** del item.

- 1 item de 400 KB = 100 RCU por lectura strongly (400/4).
- Un puñado de items grandes te puede agotar la capacidad rápido.

---

## Hot Partition vs Hot Key — la distinción sutil

Tienen el mismo síntoma (throttling) pero **origen distinto**.

| Concepto       | Origen                          | Analogía biblioteca                     |
|----------------|---------------------------------|-----------------------------------------|
| **Hot Key**    | 1 item específico muy accedido  | "Un libro taquillero" (Justin Bieber)   |
| **Hot Partition** | Partición entera saturada    | "Una estantería entera colapsada"       |

### La relación entre ellas

```
HOT KEY ────causa────▶ HOT PARTITION
   │                         │
   │ (1 item específico)     │ (la partición entera saturada)
   │                         │
   ▼                         ▼
"Justin Bieber"        "muchos items accedidos      ←── puede ser por hot key
recibe 5000 req/s      al mismo tiempo en esa            o por mal diseño de PK
                       partición"
```

> **Hot Key SIEMPRE causa Hot Partition. Pero Hot Partition no siempre es por Hot Key.**

Es como pulgares y dedos: todos los pulgares son dedos, pero no todos los dedos son pulgares.

---

## Las soluciones según la causa

### Para Hot Partition → rediseño de Partition Key

- Cambiar la PK por una con más cardinalidad (UUID, ID único).
- **Write sharding** — agregar sufijo random:
  - `date` → `date#0`, `date#1`, ..., `date#9`
  - Distribuye un volumen alto entre N particiones.

### Para Hot Key → cache (DAX), NO cambiar la PK

**Error común**: pensar que Hot Key significa "elegí mal la PK". NO necesariamente.

Ejemplo: `user_id` como PK puede ser una excelente elección con millones de usuarios. Pero si un usuario específico se vuelve viral (pensá en Messi publicando algo), ESA clave recibe el 80% de las lecturas. **La PK está bien diseñada — el problema es un patrón de acceso puntual.**

Cambiar la PK implicaría:
- Rediseñar toda la tabla
- Migrar todos los datos
- Cambiar todas las queries de la aplicación

Eso es **nuclear** y no resuelve el problema — mañana otro usuario se vuelve viral y estás igual.

**La solución correcta es DAX (DynamoDB Accelerator):**

- Cache en memoria que se pone **delante** de DynamoDB.
- Los items populares se devuelven desde RAM en **microsegundos**.
- La partición se libera porque las lecturas no la tocan.
- No cambiás NADA del modelo de datos.

> Por eso la diapo dice: *"Si hay problemas de RCU, podemos utilizar DAX"*.

### Cuándo SÍ cambiar la PK

Solo cuando el problema es **Hot Partition por mal diseño** — la PK tiene poca cardinalidad desde el inicio.

Ejemplo: usar `country_code` como PK cuando el 90% de tus usuarios son de Argentina → la partición "AR" está siempre saturada. Acá sí, el diseño de la PK es el problema.

| Problema | ¿Cambiar PK? | Solución real |
|---|---|---|
| Hot Key (usuario viral) | **NO** | DAX (cache) |
| Hot Partition (PK con poca cardinalidad) | **SÍ** | Rediseñar PK / write sharding |

### Para items grandes → rediseño de datos

- Datos pesados (imágenes, PDFs) → guardar en **S3**, solo referencia en DynamoDB.
- Patrón "Large Object Storage" (visto en `03-core-concepts.md`).

### Para cualquier caso (parche temporal) → Exponential Backoff

- El SDK de AWS lo hace **automáticamente por default**.
- Si recibe throttling, reintenta con esperas crecientes:
  - Intento 1 falla → espera 1s
  - Intento 2 falla → espera 2s
  - Intento 3 falla → espera 4s
  - Intento 4 falla → espera 8s
- Esto NO arregla la causa, solo evita que tu app se rompa mientras AWS se estabiliza.
- Aprovecha que muchos throttlings son **transitorios**.

---

## Tabla de decisión rápida

| Síntoma                                                    | Causa probable     | Solución             |
|------------------------------------------------------------|--------------------|----------------------|
| Un solo item muy popular satura la tabla                   | Hot Key            | **DAX**              |
| Una partición saturada, las demás idle                     | Hot Partition      | Rediseñar PK / sharding |
| Lecturas consumen RCU enorme aunque sean pocos requests    | Items grandes      | S3 + referencia      |
| Throttling intermitente y corto                            | Pico temporal      | Exponential backoff (ya viene) |
| Throttling sostenido                                       | Subaprovisionado o hot partition | Aumentar capacidad o rediseñar |

---

## Trampas típicas del examen

1. **"La aplicación recibe `ProvisionedThroughputExceededException` aunque la capacidad total no se agotó"** → Hot Partition.

2. **"Un producto viral está saturando la tabla"** → Hot Key → **DAX**.

3. **"Cómo manejo throttling transitorio sin cambiar arquitectura"** → Exponential backoff (ya está en el SDK).

4. **"Items grandes consumen mucha RCU"** → mover datos grandes a S3.

5. **"Cómo distribuir mejor las claves de partición"** → write sharding (sufijo random).

6. **DAX vs ElastiCache** (puede aparecer):
   - **DAX** es **específico para DynamoDB**, write-through, integrado nativamente.
   - **ElastiCache** es genérico, requiere más código manual.
   - Para DynamoDB → DAX casi siempre es la respuesta correcta.

---

## Auto-test mental

1. ¿Cuál es la diferencia entre Hot Key y Hot Partition?
2. Si un item es muy popular (Hot Key), ¿cuál es la mejor solución?
3. Si la partición entera está saturada por mal diseño de PK, ¿cuál es la solución?
4. ¿Qué es exponential backoff y dónde se implementa?
5. Si un item pesa 400 KB y lo lees con strongly consistent, ¿cuántos RCU consume?
6. ¿Hot Key siempre causa Hot Partition? ¿Y al revés?
7. ¿Para qué sirve write sharding?

### Respuestas

1. Hot Key = 1 item específico muy accedido. Hot Partition = la partición entera saturada (puede ser por múltiples items o por una hot key).
2. **DAX** (cache en memoria).
3. Rediseñar la Partition Key con más cardinalidad, o aplicar **write sharding**.
4. Reintento con esperas crecientes (1s, 2s, 4s, 8s...). Lo hace el SDK de AWS automáticamente.
5. 400 KB / 4 = 100 RCU por lectura strongly.
6. Hot Key SIEMPRE causa Hot Partition. Hot Partition NO siempre es por Hot Key.
7. Para distribuir una Partition Key con poca cardinalidad agregando un sufijo random, evitando hot partition.
