# Deployment Types — Beanstalk vs CodeDeploy

## Elastic Beanstalk — políticas

| Política                       | Downtime | Capacidad                | Costo extra   | Rollback          |
| ------------------------------ | -------- | ------------------------ | ------------- | ----------------- |
| **All at once**                | SÍ       | baja                     | No            | manual lento      |
| **Rolling**                    | No       | REDUCIDA                 | No            | manual            |
| **Rolling + additional batch** | No       | FULL                     | Sí (temporal) | manual lento      |
| **Immutable**                  | No       | FULL (instancias nuevas) | Sí (doble)    | RÁPIDO            |
| **Traffic splitting**          | No       | FULL                     | Sí            | rápido (canary %) |

### Disparadores

- "zero downtime + capacidad full + rollback más rápido + más seguro" → **Immutable**
- "probar con % de usuarios" (canary) → **Traffic splitting**
- "más barato/rápido, acepto downtime" → **All at once**

### Trampa

Rolling vs Rolling+additional batch: la diferencia NO es capacidad de rollback, es la
**capacidad durante el deploy**. Rolling baja capacidad; +batch la mantiene FULL.
Immutable gana en rollback porque usa instancias NUEVAS y separadas → terminás y listo.

## CodeDeploy — Lambda/ECS

| Config          | Qué hace                                                             |
| --------------- | -------------------------------------------------------------------- |
| **Canary**      | X% ahora, resto tras N min (2 saltos). Ej: `Canary10Percent5Minutes` |
| **Linear**      | X% cada N min (parejo hasta 100%)                                    |
| **All-at-once** | todo de golpe                                                        |

### Trampa por plataforma

- **EC2** con CodeDeploy: **In-place** vs **Blue/Green** (NO Canary/Linear).
- **Lambda/ECS**: **Canary / Linear / All-at-once** (NO In-place).
- Si la pregunta dice Lambda y ves "In-place" → distractor (y viceversa).

## AppSpec

- EC2/on-prem: `appspec.yml` (YAML) en la RAÍZ del bundle.
- Lambda/ECS: `appspec.yml` o `.json`.
- Hooks EC2: BeforeInstall → AfterInstall → ValidateService (último = health checks).
- Hooks Lambda/ECS: BeforeAllowTraffic / AfterAllowTraffic (NO BeforeInstall).

### Anti-trampa hooks (el nombre ES la respuesta)

- "validar ANTES de mandar tráfico" → **BeforeAllowTraffic**
- "validar/notificar DESPUÉS del shift" → **AfterAllowTraffic**
- Before/After + AllowTraffic = literal. No lo razones de más, leelo.
- Subrayar mentalmente: ANTES / DESPUÉS / NO / EXCEPTO / SOLO → invierten la respuesta.

## Patrón de error propio

NO elegir "lo más seguro/caro" por reflejo. Leer QUÉ piden optimizar
(costo / simplicidad / velocidad / seguridad) y elegir en base a ESO.

## Pregunta de prueba

App en Elastic Beanstalk: necesitás actualizar con **cero downtime, capacidad
completa todo el tiempo y el rollback más rápido posible**. ¿Qué política?

A) All at once
B) Rolling
C) Rolling with additional batch
D) Immutable

<details><summary>Respuesta</summary>

**D** (Immutable): instancias NUEVAS y separadas → rollback = terminás y listo.
Cuándo sería cada una:
- **All at once** → "lo más barato/rápido y acepto downtime".
- **Rolling** → sin costo extra pero acepto capacidad reducida durante el deploy.
- **Rolling + additional batch** → capacidad FULL pero rollback lento (instala sobre existentes).
- **Immutable** → zero downtime + full + rollback rápido (lo más seguro).
- **Traffic splitting** → "probar con un % de usuarios" (canary).
</details>
