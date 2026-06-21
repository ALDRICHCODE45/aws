# EventBridge vs CloudWatch Alarm vs CloudTrail (quién dispara qué)

## El trío que confunde
| Necesidad | Servicio |
| --------- | -------- |
| "cuando PASA un evento en AWS → ejecutar algo" | **EventBridge rule** (event pattern) |
| "cuando una MÉTRICA cruza un umbral" (CPU>80%) | **CloudWatch Alarm** |
| "auditar QUIÉN hizo qué llamada API" | **CloudTrail** (logging, NO trigger) |

## Regla de oro
"Reaccionar a un evento de AWS + Lambda + eficiente/pocos componentes"
→ **EventBridge** SIEMPRE.
EventBridge = pegamento event-driven: sirve para eventos REACTIVOS (algo pasó)
y PROGRAMADOS (cron / scheduled).

## Caso típico (EC2 launch → Lambda → DynamoDB)
- "cada vez que se lanza una EC2, ejecutar Lambda, eficiente, sin componentes extra"
  → **EventBridge rule** con event pattern "EC2 Instance State-change" → Lambda target.

## Por qué NO las otras
- ❌ **CloudTrail + alarma** → CloudTrail solo AUDITA (no dispara). Cadena larga:
  CloudTrail→Logs→metric filter→alarma→SNS→Lambda = muchos componentes.
  Si el enunciado dice "sin componentes adicionales / sin monitoreo innecesario"
  → esa frase es la PISTA para descartar CloudTrail y alarmas.
- ❌ **CloudWatch Alarm** → es para umbrales de MÉTRICAS, no para "esta instancia
  se lanzó". Además no invoca Lambda directo (va a SNS primero).

## Pista de lectura
"más eficiente / sin componentes adicionales / sin monitoreo innecesario" =
te están empujando a EventBridge (la solución nativa de 1 pieza).

## Pregunta de prueba

Cada vez que se lanza una EC2 querés ejecutar una Lambda que registre metadata,
de la forma más eficiente y sin componentes adicionales. ¿Qué hacés?

A) Configurar CloudTrail para registrar RunInstances y activar la Lambda con una alarma
B) Crear una regla de EventBridge que detecte el cambio de estado de EC2 y apunte a la Lambda
C) Crear una alarma de CloudWatch basada en métricas de inicialización de EC2
D) Crear una alarma de CloudWatch Logs que detecte cambios de estado

<details><summary>Respuesta</summary>

**B** (EventBridge rule → Lambda): nativo, 1 pieza, en tiempo real.
Cuándo sería cada una:
- **CloudTrail** → para AUDITAR quién hizo qué API (no es un disparador; cadena larga).
- **CloudWatch Alarm** → cuando una MÉTRICA cruza un umbral (CPU>80%), no "se lanzó una EC2".
</details>
