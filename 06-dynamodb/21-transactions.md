# DynamoDB — Transactions

Operaciones de todo o nada en varios ítems de una o varias tablas. Proporciona ACID.

---

## APIs

- **TransactGetItems** → una o más operaciones GetItem
- **TransactWriteItems** → una o más operaciones PutItem, UpdateItem, DeleteItem

## Costo: ×2

Consume **2x WCU y 2x RCU** porque DynamoDB hace 2 fases por ítem: preparar + confirmar.

## Cálculos de capacidad (importante para examen)

**Escritura transaccional:** 3 escrituras/s, ítem de 5 KB
```
3 × ceil(5/1) × 2 = 30 WCU
```

**Lectura transaccional:** 5 lecturas/s, ítem de 5 KB
```
5 × ceil(5/4) × 2 = 5 × 2 × 2 = 20 RCU
```
(5 KB se redondea a 8 KB → 2 bloques de 4 KB)

## Regla rápida

> Es la fórmula normal de WCU/RCU pero **×2 al final**.

## Casos de uso

Transacciones financieras, gestión de pedidos, juegos multijugador — cualquier caso donde "o pasan todas las operaciones o no pasa ninguna".
