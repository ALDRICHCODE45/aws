# DynamoDB — Optimistic Locking (Bloqueo Optimista)

Usa Conditional Writes + un atributo de versión para evitar que dos escrituras concurrentes se pisen.

---

## Cómo funciona

1. Cada ítem tiene un atributo `version`
2. Al escribir, la condición dice: "solo si version = X"
3. Si pasa → escribe y sube la versión a X+1
4. Si otro ya escribió antes → version cambió → condición falla → `ConditionalCheckFailedException`

## Dos escrituras "al mismo tiempo"

DynamoDB **serializa** las escrituras a nivel de ítem. Siempre hay un primero y un segundo:

- Primero → condición pasa → escribe → version sube
- Segundo → condición falla → rechazado → puede reintentar con la versión nueva

## Caso de uso

Dos usuarios editando el mismo perfil, dos procesos actualizando stock, dos Lambdas procesando el mismo pedido. Sin versión el último pisa al primero silenciosamente.

## Para el examen

- "Evitar que dos escrituras concurrentes se sobreescriban" → **Optimistic Locking** con Conditional Writes
- NO es un lock real (eso sería pesimista) — todos pueden intentar, solo gana el primero
- El que pierde recibe `ConditionalCheckFailedException`, no se queda esperando

## Optimistic vs Pessimistic (trampa de examen)

- **Optimistic** = sin bloqueo, valida versión al escribir → rendimiento NO se afecta.
  DynamoDB lo soporta nativamente (conditional writes + version attribute).
- **Pessimistic** = candado/lock mientras procesás → los demás ESPERAN → MATA
  rendimiento bajo alta concurrencia. DynamoDB NO lo soporta nativamente.
- Keyword: "sin afectar rendimiento" + "race condition" → siempre **optimistic**.
- "Bloqueo optimista LAXO / sin validación" → no previene overwrites. Distractor.

## Pregunta de prueba

Una app Java con alta concurrencia ve cómo escrituras concurrentes se pisan
(race conditions) sobrescribiendo datos recientes. ¿Mejor estrategia SIN afectar
significativamente el rendimiento?

A) Bloqueo pesimista con candados de escritura prolongados
B) Bloqueo optimista con control de versiones (atributo de versión)
C) Bloqueo pesimista mediante lectura exclusiva
D) Control de versiones sin validación explícita (optimista laxo)

<details><summary>Respuesta</summary>

**B** (optimistic locking con version attribute): sin bloqueo → no afecta rendimiento.
Cuándo sería cada una:
- **pesimista (A/C)** → los demás esperan = mata rendimiento; DynamoDB no lo soporta nativo.
- **optimista laxo sin validación (D)** → sin validar la versión no previene los overwrites.
</details>
