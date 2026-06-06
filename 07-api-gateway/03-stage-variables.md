# API Gateway — Stage Variables (Variables de Etapa)

## Qué son

**Variables key-value** que viven en API Gateway, asociadas a un Stage específico. Funcionan como **variables de entorno PARA API Gateway** (NO para Lambda).

> Formato de uso: `${stageVariables.nombreVariable}`

## Diferencia crítica vs Lambda Environment Variables

Esto es trampa clásica de examen:

| Lambda env vars                                   | API Gateway stage variables                              |
| ------------------------------------------------- | -------------------------------------------------------- |
| Viven **dentro** de Lambda                        | Viven en **API Gateway**                                 |
| Las lee el **código** de Lambda (`process.env.X`) | Las resuelve **API Gateway ANTES** de invocar el backend |
| Cambiarlas requiere **redeploy** de Lambda        | Cambiarlas es **inmediato**, sin redeploy                |
| Privadas a la Lambda                              | Compartidas entre todas las integraciones del stage      |

> Regla mental: si la variable la usa **API Gateway para decidir A QUIÉN llamar** → stage variable. Si la usa el **código del backend para configurarse** → env var del backend.

## Dónde se pueden usar

Las stage variables se pueden inyectar en **3 lugares**:

### 1. ARN de función Lambda (lo más común)

Para apuntar a **diferentes versiones/aliases de Lambda** según el stage:

```
arn:aws:lambda:us-east-1:123:function:miFuncion:${stageVariables.alias}
```

- Stage `dev` → variable `alias=dev` → llama a `miFuncion:dev`
- Stage `prod` → variable `alias=prod` → llama a `miFuncion:prod`

### 2. Endpoint HTTP

Para apuntar a **diferentes URLs de backend** según el stage:

```
https://${stageVariables.backendUrl}/api/users
```

- Stage `dev` → variable `backendUrl=api.dev.miapp.com` → llama al backend de dev
- Stage `prod` → variable `backendUrl=api.miapp.com` → llama al backend de prod

### 3. Mapping Templates (plantillas de mapeo de parámetros)

Para pasar valores como **parámetros al backend** (vía mapping templates de VTL):

```text
{
  "ambiente": "$stageVariables.environment",
  "logLevel": "$stageVariables.logLevel"
}
```

## Stage Variables y Lambda Context

Cuando API Gateway invoca a Lambda, las stage variables se pasan **dentro del objeto `event`** que recibe la Lambda:

```javascript
exports.handler = async (event) => {
  const stageVars = event.stageVariables;
  // stageVars.alias, stageVars.environment, etc.
};
```

> Truco de examen: la diapo dice "se pasan al objeto contexto en Lambda" — en realidad llegan en `event.stageVariables`, NO en el `context` de Lambda. Es un detalle, pero AWS suele decir "context" en sentido amplio.

## Caso de uso CLAVE — Lambda Aliases + Stage Variables

El patrón canónico para multi-ambiente:

1. Lambda con aliases: `miFuncion:dev`, `miFuncion:test`, `miFuncion:prod`.
2. API Gateway con un SOLO deployment, integración apuntando a:
   `arn:aws:...:miFuncion:${stageVariables.lambdaAlias}`
3. Stage `dev` con variable `lambdaAlias=dev`.
4. Stage `prod` con variable `lambdaAlias=prod`.
5. **El MISMO deployment** funciona en ambos stages, llamando a Lambdas distintas.

> Esto es la respuesta correcta cuando el examen pregunta: "¿Cómo hago que la misma API apunte a Lambdas distintas según el ambiente?"

## Caso de uso 2 — Endpoints HTTP por ambiente

Mismo patrón pero con backends HTTP:

- Stage `dev` → `backendHost=api.dev.empresa.com`
- Stage `prod` → `backendHost=api.empresa.com`
- Integración: `https://${stageVariables.backendHost}/users`

## Trampas de examen

1. **Stage variable ≠ Lambda env var**. Stage variable la resuelve API Gateway; env var la lee el código de Lambda.
2. Cambiar stage variable es **inmediato**, no requiere redeploy de API ni de Lambda.
3. En Lambda llegan en `event.stageVariables`, NO en `context`.
4. Sirven para: ARN de Lambda, URL de HTTP backend, mapping templates. **NO** para cambiar throttling, cache, ni config del stage (esas son config del stage, no variables).
5. Patrón ganador multi-ambiente: **stage variables + Lambda aliases** = mismo deployment, distintos targets.

## Auto-test

1. ¿Cuál es la diferencia entre una stage variable de API Gateway y una env var de Lambda?
2. Querés que el MISMO deployment de API Gateway apunte a `miFuncion:dev` en el stage dev y a `miFuncion:prod` en el stage prod. ¿Cómo lo configurás?
3. Cambiás el valor de una stage variable en prod. ¿Necesitás redeployar la API?
4. Dentro de Lambda, ¿en qué objeto recibís el valor de las stage variables?
5. ¿Podés usar stage variables para cambiar el nivel de throttling de un stage?

<details>
<summary>Respuestas</summary>

1. La **stage variable** la resuelve API Gateway ANTES de invocar el backend (sirve para decidir A QUIÉN llamar). La **env var de Lambda** vive dentro de Lambda y la lee el código de la función. Cambiar stage variable es inmediato; cambiar env var requiere update de Lambda.
2. Configurás la integración apuntando a `arn:aws:...:miFuncion:${stageVariables.alias}`. En el stage dev creás la variable `alias=dev`; en prod creás `alias=prod`. Mismo deployment, distintos targets.
3. **No**. Stage variables se aplican en runtime. El cambio es inmediato.
4. En `event.stageVariables` (NO en `context`, aunque la diapo lo llame así de forma genérica).
5. **No**. Throttling es configuración del stage, no se controla con stage variables. Las variables sirven para ARN de Lambda, URL HTTP, y mapping templates.

</details>
