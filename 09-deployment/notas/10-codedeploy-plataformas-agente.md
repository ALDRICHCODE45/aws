# CodeDeploy — plataformas, agente y deployment types

## Plataformas de cómputo
- **EC2 / On-premises** (servidores locales SÍ soportados)
- **AWS Lambda**
- **Amazon ECS**
(Decir "solo EC2/Lambda/ECS" es FALSO: falta on-premises.)

## Agente de CodeDeploy
- Requerido SOLO en **EC2 / on-premises**.
- **NO** se instala en Lambda ni en ECS (esas son gestionadas).
- Se comunica por **HTTPS (443)**, NO HTTP/80.

## Deployment types por plataforma
| Plataforma | In-place | Blue/Green |
| ---------- | -------- | ---------- |
| EC2/on-prem | ✅ | ✅ |
| Lambda | ❌ | ✅ (solo) |
| ECS | ❌ | ✅ (solo) |

- **In-place = SOLO EC2/on-premises.**
- **Lambda y ECS = SOLO blue/green** (traffic shifting), nunca in-place.

## Trampas (todas cayeron en una pregunta)
- "agente en EC2 Y ECS" → FALSO (ECS no usa agente).
- "agente por HTTP/80" → FALSO (HTTPS/443).
- "CodeDeploy solo EC2/Lambda/ECS" → FALSO (falta on-premises).
- "Lambda puede in-place" → FALSO (solo blue/green).

## Ganchos
Agente = solo EC2/on-prem. In-place = solo EC2/on-prem. Lambda/ECS = solo blue/green.

## Pregunta de prueba

¿Cuáles son consideraciones VÁLIDAS al usar CodeDeploy? (Elegí DOS)

A) CodeDeploy solo puede desplegar en EC2, Lambda y ECS
B) CodeDeploy puede desplegar en EC2 y en servidores on-premises
C) Las implementaciones en Lambda no pueden usar in-place
D) Debés instalar el agente de CodeDeploy en EC2 y en clústeres ECS

<details><summary>Respuesta</summary>

**B y C**.
Cuándo sería cada una (por qué las otras son falsas):
- **A** falso → falta on-premises (el "solo" la mata).
- **D** falso → el agente es SOLO para EC2/on-premises; ECS y Lambda NO usan agente.
- In-place existe SOLO en EC2/on-premises; Lambda y ECS son solo blue/green.
</details>
