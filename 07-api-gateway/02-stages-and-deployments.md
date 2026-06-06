# API Gateway — Stages (Etapas de Despliegue)

## La regla de oro

> **Hacer cambios en API Gateway NO los hace efectivos. Necesitás "desplegarlos" (deploy) para que tengan efecto.**

Esto es **fuente clásica de confusión**. Vos editás rutas, métodos, integraciones — y nada cambia en producción hasta que hacés un **deployment**.

## Qué es un Stage

Un **Stage** es un **ambiente** con nombre (dev, test, staging, prod — el nombre lo elegís vos) que apunta a una **versión congelada** de tu API.

- Podés tener **TANTOS stages como quieras**.
- Cada stage tiene **SU PROPIA URL** — por ejemplo:
  - `https://abc123.execute-api.us-east-1.amazonaws.com/dev`
  - `https://abc123.execute-api.us-east-1.amazonaws.com/prod`
- Cada stage tiene **su propia configuración** independiente: variables de entorno, throttling, logging, caching, etc.

## Qué es un Deployment

Un **Deployment** es un **snapshot** (foto congelada) de tu API en un momento dado. Cuando hacés "Deploy API" elegís a qué stage va ese snapshot.

> Analogía: pensá en **stages como cajones** (dev, prod) y **deployments como fotos** que metés adentro del cajón. Cada cajón muestra la foto más reciente que le pusiste.

## Flujo típico

1. Editás tu API (agregás un endpoint nuevo, cambiás una integración).
2. **NADA cambia todavía** en ningún ambiente.
3. Hacés "Deploy API" → elegís el stage `dev`.
4. Probás en la URL de `dev`. Si está OK:
5. Hacés OTRO "Deploy API" → elegís el stage `prod`.
6. Ahora prod tiene los cambios.

> **Importante**: NO "promovés" código de dev a prod. Hacés un deployment nuevo apuntando a prod. El concepto es distinto a CI/CD tradicional.

## Rollback — la parte buena

API Gateway **guarda el historial de deployments** por stage. Si rompiste algo en prod:

1. Vas a "Deployment History" del stage `prod`.
2. Elegís un deployment anterior que funcionaba.
3. Apuntás el stage a ese deployment.
4. **Rollback inmediato**, sin redeploy de código.

> Esto es **muy poderoso** y muy preguntado en examen: rollback en API Gateway = cambiar el deployment al que apunta el stage.

## Stage Variables (bonus importante)

Cada stage puede tener **variables** (key-value pairs) que se inyectan en runtime. Sirven para:

- Apuntar a **diferentes versiones de Lambda** según el stage (dev → Lambda:dev, prod → Lambda:prod).
- Apuntar a **diferentes endpoints HTTP** por ambiente.
- Pasar config al backend sin hardcodearla.

Ejemplo: variable `lambdaAlias` = `dev` en stage dev, y `prod` en stage prod. La integración usa `mi-lambda:${stageVariables.lambdaAlias}`.

> **Truco de examen**: stage variables + Lambda aliases = forma canónica de tener "el mismo deployment de API" apuntando a Lambdas diferentes según ambiente.

## Canary Deployments (avanzado, viene en otra clase pero menciono)

Podés hacer un deployment **canary** en un stage: un % del tráfico va a la versión nueva, el resto sigue en la vieja. Para ir probando gradualmente. Lo vemos más a fondo cuando aparezca en el curso.

## Trampas de examen

1. **"Hice un cambio y no se ve"** → te falta hacer **deployment**. Los cambios NO son efectivos hasta deployar.
2. **Cada stage tiene su URL** — `/dev`, `/prod` son partes de la URL, NO subdominios.
3. **Rollback = cambiar el deployment activo del stage**, NO redeployar código viejo.
4. **Stage variables se resuelven en runtime** — útiles para apuntar a recursos distintos por ambiente.
5. **No hay "promoción" automática** entre stages — cada deployment es explícito y apunta a UN stage.
6. Cada stage tiene **su propia config** de throttling, logging, caching, X-Ray, WAF, etc. — NO se heredan entre stages.

## Auto-test

1. Hacés un cambio en una ruta de tu API Gateway y refrescás Postman pero sigue devolviendo lo viejo. ¿Qué te falta?
2. Rompiste prod con un deployment nuevo. ¿Cómo hacés rollback rápido sin reescribir código?
3. Querés que la MISMA API apunte a `mi-lambda:dev` en el stage dev y a `mi-lambda:prod` en el stage prod. ¿Qué usás?
4. ¿Podés tener throttling distinto en dev y prod aunque sea la misma API?
5. ¿La URL del stage prod y dev son subdominios distintos?

<details>
<summary>Respuestas</summary>

1. **Hacer un deployment** apuntando al stage donde querés ver el cambio. Editar no es desplegar.
2. Ir al **Deployment History** del stage prod y apuntar el stage a un deployment anterior que funcionaba. Rollback inmediato.
3. **Stage variables** + Lambda aliases. Variable tipo `lambdaAlias` con valor distinto por stage, y la integración usa `mi-lambda:${stageVariables.lambdaAlias}`.
4. **Sí**. Cada stage tiene su propia configuración independiente de throttling, caching, logging, etc.
5. **No**. Son **paths** dentro de la misma URL: `.../dev` y `.../prod`. Para tener subdominios reales necesitás **custom domain names**.

</details>
