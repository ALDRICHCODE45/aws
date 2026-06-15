# API Keys vs Lambda Authorizer para límites de uso

## El error mental

Pensar que "necesito autenticar + limitar requests" → siempre es **Cognito + API Keys con Usage Plans**.

**MAL.** Los Usage Plans aplican el límite **a la API Key**, no al usuario. No escalan a usuarios individuales.

## Regla de decisión

| Situación | Solución |
| --------- | -------- |
| Tiers comerciales (Free / Pro / Enterprise) | **API Keys + Usage Plans** |
| Límite por usuario individual de una app | **Lambda Authorizer + DynamoDB/Redis** |
| Solo autenticar usuarios de app móvil/web | **Cognito User Pools** (sin API Keys) |

## Por qué API Keys NO escala "por usuario"

1 usuario = 1 API Key = 1 Usage Plan asociado.
Con 1000 usuarios → **1000 API Keys + 1000 Usage Plans**. Inviable.

API Keys son para **decenas de clientes B2B**, no para millones de end users.

## Palabras clave del enunciado

| Si dice… | NO es API Keys |
| -------- | -------------- |
| "millones de usuarios" | ❌ |
| "por usuario individual" | ❌ |
| "usuarios de app móvil" | ❌ |
| "end users" | ❌ |

| Si dice… | SÍ es API Keys |
| -------- | -------------- |
| "clientes comerciales" / "B2B" | ✅ |
| "monetizar la API" | ✅ |
| "Free / Pro / Enterprise tiers" | ✅ |
| "third-party developers / partners" | ✅ |

## Aclaración importante

**Lambda Authorizer hace AMBAS: autentica Y autoriza.** El nombre engaña. Si la pregunta dice "autenticar usuarios con un IdP custom (Auth0, Okta, JWT propio)" → Lambda Authorizer es la respuesta, no Cognito.
