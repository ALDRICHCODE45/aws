# Cuándo GSI vs LSI

- **Necesitás consultar por un atributo que NO es la PK actual** → **GSI** (PK nueva)
- **Necesitás ordenar diferente DENTRO de la misma PK** → **LSI** (misma PK, otra SK)

## Ejemplo

Tabla: PK = `user_id`, SK = `order_date`

- "Pedidos del usuario X ordenados por producto" → LSI podría servir
- "TODOS los pedidos del producto Y sin importar el usuario" → solo **GSI** con PK = `product_id`

## Regla rápida

> LSI = otro orden para la misma partición.
> GSI = otra forma de particionar los datos.
