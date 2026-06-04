# DynamoDB — TTL (Time To Live)

Elimina ítems automáticamente después de una fecha de caducidad.

---

## Cómo funciona

- El atributo TTL debe ser tipo **Number** con valor **Unix Epoch timestamp**
- Cuando el timestamp expira, DynamoDB marca el ítem como caducado
- Se elimina físicamente dentro de las **48 horas** siguientes
- Durante esas 48h el ítem **sigue apareciendo** en lecturas — filtrarlo es tu responsabilidad
- Se borra de la tabla base, LSI y GSI

## Costo

- **NO consume WCU** — es gratis
- Trampa: "eliminar datos viejos sin costo adicional" → TTL

## Integración con Streams

- Cada eliminación por TTL genera un registro en DynamoDB Streams
- Permite recuperar elementos eliminados o disparar acciones (notificaciones, auditoría)

## Casos de uso

- Sesiones de usuario expiradas
- Reducir almacenamiento (solo datos actuales)
- Cumplir normativas de retención de datos (GDPR, etc.)
