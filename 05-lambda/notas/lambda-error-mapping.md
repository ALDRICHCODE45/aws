# Lambda — Mapeo de errores

> Si la pregunta menciona el **nombre exacto del error** → es de mapeo, NO de razonamiento.

| Error | Causa real |
| ----- | ---------- |
| `InvalidParameterValueException` | **Rol IAM que Lambda no puede asumir** (trust policy mal) |
| `ResourceConflictException` | Recurso YA existe (nombre duplicado) |
| `CodeStorageExceededException` | Superaste **75 GB** de código en la región |
| `ServiceException` | Error interno de AWS → **retry** |
| `TooManyRequestsException` | Throttling de invocaciones |
| `RequestEntityTooLargeException` | Payload > 6 MB (sync) / 256 KB (async) |
| `ResourceNotFoundException` | Función/alias/versión inexistente |
| `EC2ThrottledException` | Throttling de ENIs en VPC |

## Trampa clásica

`InvalidParameterValueException` al deployar → **trust policy del rol sin `lambda.amazonaws.com` como Principal**. El rol existe pero Lambda no lo puede asumir.
