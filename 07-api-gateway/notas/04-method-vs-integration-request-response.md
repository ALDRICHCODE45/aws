# API Gateway — Los 4 cajones (Method vs Integration)

## Flujo
```
Cliente → [Method Request] → [Integration Request] → Backend (Lambda)
Cliente ← [Method Response] ← [Integration Response] ← Backend
```

## Qué configura cada uno
| Cajón | Lado | Configurás |
| ----- | ---- | ---------- |
| **Method Request** | cliente → API GW | qué DEBE mandar el cliente: **query strings, headers, validación, auth** |
| **Integration Request** | API GW → backend | **mapeo/transformación** hacia Lambda (mapping templates VTL) |
| **Integration Response** | backend → API GW | transformar respuesta del backend |
| **Method Response** | API GW → cliente | status codes y headers devueltos al cliente |

## Regla de oro
- **"Method" = lado CLIENTE** (qué manda/recibe el cliente).
- **"Integration" = lado BACKEND** (cómo se mapea hacia/desde Lambda).

## Pregunta tipo (error simulacro #2)
"los consumidores DEBEN incluir un query param `tipoCurso`"
→ declarar query string = lado cliente = **Method Request** (URL Query String Parameters).
NO Integration Request (eso es donde lo mapeás al backend, paso posterior).

## Gancho
"el cliente debe enviar X (query/header/validación)" → Method Request.
"transformar/mapear hacia o desde el backend" → Integration Request/Response.

## Pregunta de prueba

En una integración Lambda no-proxy, los consumidores deben incluir un query param
`tipoCurso`. ¿Dónde lo configurás para declararlo?

A) Integration Request
B) Method Request
C) Method Response
D) Integration Response

<details><summary>Respuesta</summary>

**B** (Method Request): declarar lo que el CLIENTE debe enviar (query strings, headers).
Cuándo sería cada una:
- **Integration Request** → mapear ese parámetro hacia el evento de Lambda (paso posterior).
- **Method Response** → status/headers que se devuelven al cliente.
- **Integration Response** → transformar la respuesta del backend.
</details>
