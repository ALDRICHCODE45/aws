# API Gateway — Canary Deployments

## De dónde viene el nombre

Los mineros del siglo XIX llevaban un **canario** a la mina. Si había gas tóxico, el canario moría primero y avisaba a los mineros antes de que les pasara a ellos. **Era el sistema de alerta temprana.**

En software es lo MISMO: mandás un **% pequeño de tráfico real** (5%, 10%) a la versión nueva. Si algo explota, afecta solo a ese % y te enterás **antes** de hacer el rollout completo.

## Qué es

Un **deployment canary** te permite tener **DOS versiones de tu API conviviendo en el MISMO stage**, repartiendo el tráfico entre ellas según un porcentaje que vos definís.

- **Versión estable** (ej: v1) → recibe la mayoría del tráfico (95%).
- **Versión canary** (ej: v2) → recibe un % chico (5%) para probar en producción real.

> **OJO**: el canary vive **dentro del mismo stage** (típicamente prod). NO son dos stages distintos. Es UN stage manejando dos snapshots simultáneos.

## Cómo funciona

1. Tenés el stage `prod` corriendo el deployment v1 (estable).
2. Habilitás canary en `prod` y desplegás v2 como canary.
3. Configurás: "5% del tráfico va al canary, 95% al estable".
4. API Gateway **reparte requests automáticamente** según ese %.
5. Mirás métricas y logs **del canary por separado**.
6. Si todo OK → **promovés** el canary (pasa a ser el nuevo estable y el canary desaparece).
7. Si algo explota → **descartás** el canary (todo el tráfico vuelve al 100% estable).

## Por qué es "blue/green con Lambda + API Gateway"

La diapo dice esto explícitamente — es **importante para el examen**:

- **Blue/Green deployment** tradicional = dos entornos completos (blue = viejo, green = nuevo) y switcheás tráfico entre ellos.
- En el mundo serverless (Lambda + API Gateway), el canary deployment **ES la implementación de blue/green**:
  - "Blue" = versión estable de la Lambda + integración estable de API Gateway.
  - "Green" = versión canary de la Lambda + integración canary de API Gateway.
  - El "switch" gradual de tráfico (5% → 25% → 50% → 100%) es lo que reemplaza al switch instantáneo del blue/green clásico.

> Trampa de examen: si te preguntan "cómo hago blue/green con Lambda y API Gateway" → la respuesta es **canary deployments**.

## Features importantes

### 1. Métricas y logs SEPARADOS

CloudWatch te muestra métricas del **canary aparte** de las del estable. Así detectás si la versión nueva tiene más errores, más latencia, etc. **SIN esto, el canary no serviría** — necesitás ver diferencias.

### 2. Stage Variables override

Podés tener **stage variables distintas para el canary**. Ejemplo:

- Estable: `lambdaAlias=prod` → llama a `miFuncion:prod` (v1).
- Canary: `lambdaAlias=canary` → llama a `miFuncion:canary` (v2).

Esto te permite que el canary apunte a una versión distinta de la Lambda sin tocar la estable.

### 3. Promote o Discard

- **Promote**: el canary se convierte en el nuevo estable. Limpia el canary.
- **Discard / Delete**: descartás el canary. 100% del tráfico vuelve al estable.

## Casos de uso reales

- Probar una **feature riesgosa** con tráfico real antes de full rollout.
- **Detectar bugs de performance** que no aparecen en staging por falta de carga.
- Validar cambios de **integración** (DB schema, Lambda nueva, third-party).
- **Reducir blast radius** — si rompés, rompés para 5% no para 100%.

## Canary vs Stages — la confusión clásica

| Concepto                      | Para qué sirve                                                     |
| ----------------------------- | ------------------------------------------------------------------ |
| **Stages** (dev, test, prod)  | Separar **ambientes** distintos con configs y URLs distintas       |
| **Canary dentro de un stage** | Probar una **versión nueva en el MISMO ambiente** con tráfico real |

> No son alternativas, son **complementarios**: tenés stages para ambientes, y dentro de prod podés tener canary para probar nuevas versiones gradualmente.

## Trampas de examen

1. **"Blue/Green con Lambda + API Gateway"** = canary deployment. Memorizalo así.
2. Canary vive **dentro del mismo stage**, NO es un stage aparte.
3. Métricas y logs del canary son **separados** del estable → eso es lo que permite decidir si promovés o descartás.
4. Stage variables se pueden **overridear para el canary** → útil para apuntar a una Lambda distinta.
5. Promover canary = pasa a ser el nuevo estable. Descartarlo = tráfico vuelve 100% al estable.
6. El % de tráfico al canary lo **elegís vos** (5%, 10%, 50% — lo que quieras).

## Auto-test

1. ¿Por qué se llama "canary deployment"?
2. Querés hacer blue/green deployment con Lambda + API Gateway. ¿Qué feature usás?
3. ¿El canary deployment es un stage separado dentro de API Gateway?
4. ¿Cómo hacés para que el canary llame a una versión distinta de la Lambda que la versión estable?
5. Notás que el canary tiene 30% más errores que el estable. ¿Qué hacés?
6. Si el canary funciona perfecto y querés que sea la nueva versión estable, ¿qué acción ejecutás?

<details>
<summary>Respuestas</summary>

1. Por los canarios que llevaban los mineros como **sistema de alerta temprana** — si el canario moría, sabían que el aire estaba tóxico antes de que les afectara. En software, el canary recibe poco tráfico y detecta problemas antes del rollout completo.
2. **Canary deployment** en API Gateway. AWS lo dice explícitamente: "es una implementación blue/green con Lambda + API Gateway".
3. **No**. Vive **dentro del mismo stage** (típicamente prod). Es el MISMO stage manejando dos snapshots con un split de tráfico.
4. Con **stage variables override del canary**. Ejemplo: `lambdaAlias=prod` en el estable, `lambdaAlias=canary` en el canary, e integración apuntando a `miFuncion:${stageVariables.lambdaAlias}`.
5. **Descartás el canary** (Discard / Delete). El 100% del tráfico vuelve al estable. Cero impacto en producción global.
6. **Promote canary**. Se convierte en el nuevo estable y el canary se limpia.

</details>
