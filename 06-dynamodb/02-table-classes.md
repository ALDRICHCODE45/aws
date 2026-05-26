# DynamoDB — Clases de Tabla (Table Classes)

DynamoDB ofrece **2 clases de tabla** que cambian la estructura de precios según el patrón de uso.

## 1. Standard (default)

- Diseñada para tablas con **acceso frecuente**.
- **Storage** → más caro.
- **Lectura/escritura** → más barata.
- Es la opción por defecto al crear una tabla.

## 2. Standard-IA (Infrequent Access)

- Diseñada para tablas que **guardan mucho dato pero se accede poco**.
- **Storage** → ~60% más barato que Standard.
- **Lectura/escritura** → ~25% más caro que Standard.
- Útil para: logs históricos, datos de auditoría, backups consultables, datos viejos que casi no tocás pero hay que conservar.

## Comparación rápida

| Aspecto       | Standard          | Standard-IA       |
|---------------|-------------------|-------------------|
| Storage       | Más caro          | **Más barato**    |
| Lectura       | **Más barata**    | Más cara          |
| Escritura     | **Más barata**    | Más cara          |
| Caso de uso   | Acceso frecuente  | Acceso poco frecuente |

## Analogía

- **Standard** = local en pleno centro → alquiler caro (storage), pero entra gente todo el día y vendés mucho (lecturas baratas).
- **Standard-IA** = depósito en las afueras → alquiler barato (storage), pero cada viaje a buscar algo te cuesta más (lecturas caras).

## Regla mental para examen

> "¿Accedo seguido? **Standard**.
> ¿Lo guardo más de lo que lo leo? **Standard-IA**."

## Trampa típica del examen

Si la pregunta menciona:
- "datos de auditoría que rara vez se consultan" → **Standard-IA**
- "logs históricos guardados por compliance" → **Standard-IA**
- "tabla de sesiones de usuario activa" → **Standard**
- "carrito de compras en tiempo real" → **Standard**
