# DynamoDB — Consistencia de Lectura

DynamoDB ofrece **3 niveles de consistencia** para lecturas. La elección afecta **costo + garantía** de ver el dato actualizado.

---

## Por qué existen distintos niveles

DynamoDB replica cada item en **3 servidores físicos** (3 AZs distintas) para alta disponibilidad. Cuando hacés una escritura:

1. El servidor **líder** recibe la escritura y la confirma.
2. Replica a los otros 2 servidores.
3. **La replicación tarda — típicamente <1 segundo, pero NO instantáneo.**

Si tu lectura cae en una réplica que aún no recibió el cambio → leés dato viejo.

---

## 1. Eventually Consistent Read (default)

### Cómo funciona

- La lectura puede ir a **cualquiera de los 3 servidores**.
- Si justo cae en una réplica desactualizada → devuelve dato viejo.
- *Eventualmente* todas las réplicas convergen, pero "eventualmente" no es "ahora".

### Costo

- **0.5 RCU** por lectura de hasta 4 KB.
- Es la mitad de strongly consistent.

### Cuándo usar

- Datos no críticos: feeds, listados, dashboards, búsquedas.
- Lectura donde unos milisegundos de delay no importan.
- Cuando querés **minimizar costo**.

---

## 2. Strongly Consistent Read

### Cómo funciona

- Fuerza la lectura a ir al servidor **líder** (el que recibe las escrituras primero).
- Garantiza ver el dato **actualizado al 100%**, incluso justo después de una escritura.

### Cómo activarlo

Parámetro `ConsistentRead: true` en la llamada API. Aplicable a:

- `GetItem`
- `BatchGetItem`
- `Query`
- `Scan`

**NO** se aplica a escrituras (PutItem, UpdateItem, DeleteItem) — esas son siempre consistentes por definición.

### Costo

- **1 RCU** por lectura de hasta 4 KB.
- **Doble** que eventually consistent.

### Cuándo usar

- Lectura inmediatamente después de una escritura crítica.
- Sistemas financieros, stock, autenticación.
- Cuando **NO podés permitirte** datos obsoletos.

---

## 3. Transactional Read

### Cómo funciona

- Lectura ACID de **múltiples items juntos** como una unidad atómica.
- Si una falla, **todo se revierte** (rollback).

### Costo

- **2 RCU** por lectura de hasta 4 KB.
- **El doble** que strongly consistent, **4×** que eventually.

### Cuándo usar

- Operaciones que requieren atomicidad multi-item.
- Ejemplo: leer cuenta origen + cuenta destino antes de una transferencia.

---

## Tabla resumen — costos

### Lecturas (RCU por lectura de hasta 4 KB)

| Tipo                  | Costo    |
|-----------------------|----------|
| Eventually consistent | **0.5**  |
| Strongly consistent   | **1**    |
| Transactional read    | **2**    |

### Escrituras (WCU por escritura de hasta 1 KB)

| Tipo                  | Costo    |
|-----------------------|----------|
| Standard write        | **1**    |
| Transactional write   | **2**    |

---

## Guía práctica de decisión

| Necesidad                                                | Tipo                  |
|----------------------------------------------------------|-----------------------|
| Feed, listado, búsqueda, dashboard                       | Eventually consistent |
| Lectura post-escritura crítica (stock, balance)          | Strongly consistent   |
| Operación atómica de varios items (transferencia, batch) | Transactional         |
| Lectura entre regiones (Global Tables)                   | **Solo eventually**   |

---

## Trampas típicas del examen

1. **"Default consistency"** → eventually consistent. SIEMPRE.

2. **"Read just after a write must return the latest value"** → strongly consistent.

3. **"Cost of strongly consistent vs eventually"** → strongly cuesta el **doble**.

4. **"Stale data" / "datos obsoletos" / "lag"** → eventually consistent.

5. **"Atomic multi-item operation"** → **transactional**, NO strongly consistent.

6. **"Global Tables consistency"** → entre regiones siempre es **eventually**. NO existe strongly cross-region.

7. **"ConsistentRead se puede usar en PutItem"** → falso. Solo en lecturas.

---

## Auto-test mental

1. ¿Cuánto cuesta 1 lectura eventually consistent de 4 KB?
2. ¿Cuánto cuesta 1 lectura strongly consistent de 4 KB?
3. ¿Cuánto cuesta 1 lectura transactional de 4 KB?
4. ¿Cuál es el default?
5. ¿Cómo activás strongly consistent?
6. ¿Por qué eventually consistent puede devolver datos viejos?
7. ¿En qué 4 APIs se aplica `ConsistentRead`?
8. ¿Se puede usar strongly consistent entre regiones (Global Tables)?
9. ¿Cuándo elegirías transactional read sobre strongly consistent?

### Respuestas

1. 0.5 RCU
2. 1 RCU
3. 2 RCU
4. Eventually consistent
5. Parámetro `ConsistentRead: true` en la llamada
6. Porque la replicación a las 3 AZs tarda <1s, y la lectura puede caer en una réplica atrasada
7. `GetItem`, `BatchGetItem`, `Query`, `Scan`
8. NO. Global Tables son siempre eventually consistent
9. Cuando necesitás leer **múltiples items** como unidad atómica (rollback si una falla)
