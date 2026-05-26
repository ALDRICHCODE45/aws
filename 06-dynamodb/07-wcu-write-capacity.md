# DynamoDB — WCU (Write Capacity Units)

Cálculo de capacidad de **escritura** en modo Provisioned. Cae directo en el examen como pregunta de cálculo.

---

## La fórmula en una línea

> **WCU necesarios = (escrituras por segundo) × (tamaño del item en KB, redondeado HACIA ARRIBA)**

Eso es todo. Lo demás son matices.

---

## Los 3 conceptos a internalizar

### 1. Una WCU = 1 escritura/seg de un item de hasta 1 KB

Si el item pesa 1 KB o menos → **1 WCU** por escritura/segundo.

### 2. Items más grandes consumen MÁS WCUs

Cada KB adicional (o fracción) suma 1 WCU:

| Tamaño del item | WCUs por escritura |
|-----------------|--------------------|
| 0.5 KB          | 1 WCU              |
| 1 KB            | 1 WCU              |
| 2 KB            | 2 WCUs             |
| 3 KB            | 3 WCUs             |
| 10 KB           | 10 WCUs            |

### 3. El tamaño SIEMPRE se redondea HACIA ARRIBA

DynamoDB no cobra fracciones de KB:

| Tamaño real | WCU efectivo |
|-------------|--------------|
| 0.3 KB      | 1 (mínimo)   |
| 1.01 KB     | 2            |
| 4.5 KB      | 5            |
| 7.9 KB      | 8            |

> **Regla mental:** si te pasaste de un KB por 1 byte, ya cuesta el siguiente entero.

---

## Ejemplos de la diapositiva

### Ejemplo 1 — 10 items/seg × 2 KB cada uno

```
10 escrituras/seg × 2 KB/item = 20 WCUs
```

### Ejemplo 2 — 6 items/seg × 4.5 KB cada uno

```
4.5 KB → redondea HACIA ARRIBA a 5 KB
6 escrituras/seg × 5 KB/item = 30 WCUs
```

Trampa: sin redondear da 27 (mal).

### Ejemplo 3 — 120 items/MINUTO × 2 KB cada uno

```
120/60 = 2 items/seg          ← convertir a SEGUNDOS primero
2 escrituras/seg × 2 KB/item = 4 WCUs
```

Trampa: WCU es **por segundo**, siempre normalizar el ritmo.

---

## Las 2 trampas del examen

1. **Redondear el tamaño HACIA ARRIBA siempre.**
   Item de 1.01 KB ya cuesta 2 WCUs, no 1.

2. **Convertir el ritmo a segundos.**
   Si la pregunta dice "X por minuto/hora/día" → dividir antes de multiplicar.

---

## Procedimiento mental (3 pasos)

Cuando veas una pregunta de WCU:

1. **Convertir** las escrituras a por-segundo (si vienen en otra unidad).
2. **Redondear** el tamaño del item HACIA ARRIBA al siguiente KB entero.
3. **Multiplicar** los dos números.

---

## Auto-test

Resolvé sin mirar las respuestas:

1. App escribe 50 items/seg de 1 KB cada uno → ¿WCU?
2. App escribe 8 items/seg de 3.2 KB cada uno → ¿WCU?
3. App escribe 600 items/minuto de 2 KB cada uno → ¿WCU?
4. App escribe 300 items/minuto de 3.2 KB cada uno → ¿WCU?
5. Item de 0.5 KB → ¿cuánto cuesta cada escritura?

### Respuestas

1. `50 × 1 = 50 WCUs`
2. `3.2 → 4`; `8 × 4 = 32 WCUs`
3. `600/60 = 10`; `10 × 2 = 20 WCUs`
4. `300/60 = 5`; `3.2 → 4`; `5 × 4 = 20 WCUs`
5. `1 WCU` (el mínimo, se redondea de 0.5 a 1)
