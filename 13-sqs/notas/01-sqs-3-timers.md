# SQS — Los 3 timers que confunden

| Timer | Cuándo aplica | Max |
| ----- | ------------- | --- |
| **Delivery Delay** | Mensaje **ENTRA** a la cola (oculto al agregarse) | 15 min |
| **Visibility Timeout** | Mensaje **SE LEE** (oculto a otros consumidores mientras se procesa) | 12 horas |
| **Long Polling (Receive Wait)** | Consumidor **PREGUNTA** y espera respuesta | 20 s |

## Disparadores

| Pregunta dice | Timer |
| ------------- | ----- |
| "ocultar al **agregarse / enviarse**" | **Delivery Delay** |
| "esperar antes de la primera lectura" | **Delivery Delay** |
| "dar tiempo a sistema X antes de procesar" | **Delivery Delay** |
| "evitar que **otro consumidor** procese el mismo" | **Visibility Timeout** |
| "ocultar mientras **se está procesando**" | **Visibility Timeout** |
| "consumidor espera al hacer poll" | **Long Polling** |

## Analogía del buzón

- **Delivery Delay**: cartero no pone la carta hasta X seg después → invisible porque no está.
- **Visibility Timeout**: vos sacaste la carta del buzón → invisible al resto mientras la procesás.
- **Long Polling**: te quedás parado al lado del buzón esperando una carta nueva.

## Trampa

Si confundís "ocultar mensajes" con "esperar antes de pedirlos" → caés en long polling. **El productor controla Delivery Delay. El consumidor controla Visibility Timeout y Long Polling.**
