# SQS — Short polling vs Long polling

## La idea contraintuitiva

**"Long polling" NO es lento**. Es el cliente esperando con la conexión abierta. SQS le entrega el mensaje **APENAS llega**.

|                               | Short polling          | Long polling                   |
| ----------------------------- | ---------------------- | ------------------------------ |
| Espera del cliente            | 0s (responde ya)       | Hasta 20s con conexión abierta |
| Latencia al llegar un mensaje | Espera al próximo poll | **Inmediata**                  |
| Requests vacíos               | Muchos                 | Pocos                          |
| Costo                         | Más caro               | Más barato                     |

## Disparadores

| Pregunta dice                                          | Solución                             |
| ------------------------------------------------------ | ------------------------------------ |
| "minimizar latencia" + "tráfico esporádico"            | **Long polling**                     |
| "reducir costo de API calls vacías"                    | **Long polling**                     |
| "necesito respuesta inmediata aunque no haya mensajes" | Short polling                        |
| "mensajes > 256 KB"                                    | **SQS Extended Client Library + S3** |

## Trampas

1. **El nombre "long" engaña**: la latencia de entrega es **0**, no 20s. Long se refiere a la conexión, no a la respuesta.
2. **"Reducir tamaño con compresión"** se mete como opción cuando los mensajes son chicos (4 KB). El límite es 256 KB → no hay nada que reducir.
3. **SQS Extended Client + S3** solo sirve si los mensajes superan 256 KB.

## Patrón "esporádico + baja latencia" = SIEMPRE long polling
