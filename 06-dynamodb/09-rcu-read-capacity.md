# DynamoDB — RCU (Read Capacity Units)

Cálculo de capacidad de **lectura** en modo Provisioned. Más complejo que WCU porque tiene **2 variables**: tamaño del item Y tipo de consistencia.

---

## Por qué RCU es más difícil que WCU

| WCU                            | RCU                              |
|--------------------------------|----------------------------------|
| 1 sola pregunta: tamaño        | 2 preguntas: tamaño Y consistencia |
| Bloques de **1 KB**            | Bloques de **4 KB**              |
| Sin variantes                  | Strongly vs Eventually           |

---

## Las 2 reglas fundamentales

### Regla 1 — bloques de 4 KB, redondeo HACIA ARRIBA

| Tamaño real | Bloques de 4 KB |
|-------------|-----------------|
| 1 KB        | 1               |
| 4 KB        | 1               |
| 4.1 KB      | 2               |
| 5 KB        | 2               |
| 6 KB        | 2               |
| 8 KB        | 2               |
| 9 KB        | 3               |
| 12 KB       | 3               |

> **Mental:** `bloques = ceil(tamaño_KB / 4)` — dividís entre 4 y redondeás arriba.

### Regla 2 — tipo de lectura cambia el costo por bloque

| Tipo                  | Costo por bloque de 4 KB |
|-----------------------|--------------------------|
| Strongly consistent   | **1 RCU**                |
| Eventually consistent | **0.5 RCU** (= 2 lecturas por 1 RCU) |
| Transactional read    | **2 RCU**                |

---

## Fórmulas

### Strongly consistent

```
RCU = (lecturas/seg) × (bloques de 4 KB)
```

### Eventually consistent

```
RCU = (lecturas/seg ÷ 2) × (bloques de 4 KB)
```

### Transactional read

```
RCU = (lecturas/seg × 2) × (bloques de 4 KB)
```

---

## Procedimiento mental — 4 pasos

Cuando veas una pregunta de RCU:

1. **Convertir** las lecturas a por-segundo (si vienen en minutos/horas).
2. **Calcular bloques de 4 KB** del item (dividir por 4, redondear HACIA ARRIBA).
3. **Aplicar el factor de consistencia:**
   - Strongly → multiplicar por 1 (dejar igual)
   - Eventually → dividir por 2
   - Transactional → multiplicar por 2
4. **Multiplicar** lecturas finales × bloques.

---

## Ejemplos de la diapositiva, paso a paso

### Ejemplo 1 — 10 lecturas/seg, strongly, item de 4 KB

```
Bloques: 4/4 = 1
Strongly: factor 1
10 × 1 × 1 = 10 RCU
```

### Ejemplo 2 — 16 lecturas/seg, eventually, item de 12 KB

```
Bloques: 12/4 = 3
Eventually: 16/2 = 8
8 × 3 = 24 RCU
```

### Ejemplo 3 — 10 lecturas/seg, strongly, item de 6 KB

```
Bloques: 6/4 = 1.5 → redondea a 2 (= 8 KB efectivos)
Strongly: factor 1
10 × 2 = 20 RCU
```

---

## Comparativa rápida WCU vs RCU

| Aspecto              | WCU                | RCU                                |
|----------------------|--------------------|------------------------------------|
| Tamaño de bloque     | **1 KB**           | **4 KB**                           |
| Tipos de lectura     | (no aplica)        | Strongly / Eventually / Transactional |
| Redondeo del tamaño  | Hacia arriba a 1 KB | Hacia arriba a 4 KB               |
| Conversión a segundos| Sí                 | Sí                                 |

> **Trampa clásica:** mezclar bloques entre WCU y RCU. WCU = 1 KB, RCU = 4 KB. SIEMPRE.

---

## Trampas típicas del examen

1. **"Eventually consistent"** → dividir lecturas por 2 (o equivalente: cada RCU te da 2 lecturas).
2. **"Strongly consistent"** → costo normal (1 RCU por bloque).
3. **"Transactional"** → el doble que strongly.
4. **Tamaño no múltiplo de 4** → redondear HACIA ARRIBA al siguiente múltiplo de 4 KB.
5. **Ritmo en minutos/horas** → convertir a segundos PRIMERO.
6. **No especifica tipo de consistencia** → asumir **eventually** (es el default).

---

## Auto-test (con respuestas)

1. 5 lecturas/seg, strongly, items de 4 KB → ¿RCU?
2. 10 lecturas/seg, eventually, items de 4 KB → ¿RCU?
3. 5 lecturas/seg, strongly, items de 8 KB → ¿RCU?
4. 20 lecturas/seg, eventually, items de 5 KB → ¿RCU?
5. 600 lecturas/minuto, strongly, items de 12 KB → ¿RCU?
6. 4 lecturas/seg, transactional, items de 4 KB → ¿RCU?
7. 100 lecturas/seg, eventually, items de 1 KB → ¿RCU?

### Respuestas

1. `4/4=1`; `5 × 1 = 5 RCU`
2. `4/4=1`; `(10/2) × 1 = 5 RCU`
3. `8/4=2`; `5 × 2 = 10 RCU`
4. `5→8 KB → 2 bloques`; `(20/2) × 2 = 20 RCU`
5. `600/60=10`; `12/4=3`; `10 × 3 = 30 RCU`
6. `4/4=1`; `transactional → factor 2`; `4 × 2 × 1 = 8 RCU`
7. `1→4 KB → 1 bloque`; `(100/2) × 1 = 50 RCU`
