# DynamoDB GSI — throttling que propaga a la tabla base (GOTCHA)

## El gotcha estrella del examen
- El GSI tiene su PROPIA capacidad (WCU/RCU), separada de la tabla base.
- Al escribir en la tabla base, DynamoDB también escribe en el GSI (sincronía).
- **Si el GSI está sub-aprovisionado (WCU MENOR de lo necesario) → el GSI
  se throttlea → y ese throttling PROPAGA a la TABLA BASE.**
- Resultado: las escrituras a la tabla base fallan con
  `ProvisionedThroughputExceededException`, aunque la base sola tenga capacidad.
- "Un GSI hambriento mata las escrituras de toda la tabla."

## Causa de la pregunta tipo
App con alta escritura + GSI + `ProvisionedThroughputExceeded`
→ **la WCU del GSI es MENOR que la de la tabla base** (sub-aprovisionado).

## Lógica que NO hay que invertir
`ProvisionedThroughputExceeded` = se EXCEDIÓ = FALTA capacidad = MENOR.
NUNCA "mayor" (más capacidad no causa throttling).

## Distractores
- ❌ "La tasa de solicitudes excede el rendimiento" → definición GENÉRICA del error
  (circular). Si hay opción específica (GSI), gana la específica.
- ❌ "Excede el límite de tu cuenta" → otro tipo de límite, no el caso típico.

## Regla
Específico > genérico (igual que SAM > CloudFormation).
GSI bajo de WCU = problema de TODA la tabla.

## Pregunta de prueba

Una app con alta carga de escritura y un GSI recibe `ProvisionedThroughputExceededException`.
¿Cuál es la causa más probable?

A) La WCU del GSI es MAYOR que la de la tabla base
B) La WCU del GSI es MENOR que la de la tabla base
C) El GSI tiene demasiada capacidad aprovisionada
D) Excede el límite de rendimiento de tu cuenta

<details><summary>Respuesta</summary>

**B**: GSI sub-aprovisionado se throttlea y propaga el throttling a la tabla base.
Cuándo sería cada una:
- **A / C** (más capacidad) → más capacidad nunca causa un "exceeded".
- **D** → daría otro tipo de límite/error, no el típico de un escenario con GSI.
</details>
