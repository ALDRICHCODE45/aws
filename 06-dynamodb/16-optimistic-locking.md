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
