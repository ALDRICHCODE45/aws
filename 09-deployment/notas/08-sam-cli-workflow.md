# SAM CLI — flujo de comandos (qué hace cada uno)

## Comandos
| Comando | Qué hace | Cuándo |
| ------- | -------- | ------ |
| **`sam init`** | crea proyecto SAM NUEVO (scaffold) | UNA vez, al empezar de cero |
| **`sam build`** | resuelve dependencias + construye artefactos de todas las funciones/capas | antes de cada deploy |
| **`sam package`** | sube artefactos a S3 + genera template empaquetado | (lo hace deploy hoy) |
| **`sam deploy`** | empaqueta + despliega vía **CloudFormation** | publicar, incl. Producción |
| **`sam sync`** | sincroniza cambios locales RÁPIDO (inner loop) | solo DEV/testing, **NO producción** |
| **`sam local`** | invocar/probar funciones localmente | debug local |

## Flujo para desplegar (la respuesta típica)
**`sam build` → `sam deploy`**. Dos pasos.

## Trampas (multi-select "elegí DOS")
- ❌ `sam init` cuando el dev **clonó** un repo existente → ya existe, no inicializás.
  Keyword "clona el repositorio" = NO init.
- ❌ `sam sync` para **Producción** → AWS lo desaconseja para prod (cambios directos
  sin control de deploy). Es para iteración rápida de DEV. Keyword "Producción" = NO sync.

## Meta-lección (error propio)
Estaba "seguro" de haber marcado las correctas, pero la captura mostró init+sync.
La memoria del examen NO es confiable → revisar con evidencia, no con sensación.
En el examen real: releer cada pregunta, no fiarse del "siento que voy bien".

## Pregunta de prueba

Clonaste un repo SAM existente, agregaste una función Lambda nueva y debés
desplegar a Producción. ¿Qué pasos tomás? (Elegí DOS)

A) `sam init`
B) `sam build`
C) `sam deploy`
D) `sam sync`

<details><summary>Respuesta</summary>

**B y C**: `sam build` (construye artefactos, incl. la función nueva) + `sam deploy`
(despliega vía CloudFormation).
Cuándo sería cada una:
- **sam init** → solo al crear un proyecto DE CERO (acá clonaste uno existente).
- **sam sync** → solo para iteración rápida en DEV; AWS lo desaconseja para Producción.
</details>
