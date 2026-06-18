# ECS Task Placement Strategies

Solo existen **3** estrategias reales. `azBalanced` y similares son trampa por inexistencia.

| Estrategia | Para qué |
| ---------- | -------- |
| **`binpack`** | Reducir costo / menos instancias / eficiencia CPU+memoria |
| **`spread`** | Alta disponibilidad / tolerancia a fallos / distribuir entre AZs |
| **`random`** | Testing/dev. Nunca prod. |

## Trampa cognitiva

`spread` SUENA eficiente porque "reparte parejo". **NO**: eficiencia = concentrar (binpack), no dispersar.

> Reducir costo / instancias → **binpack** (concentra)
> Tolerar fallos / HA → **spread** (dispersa)
