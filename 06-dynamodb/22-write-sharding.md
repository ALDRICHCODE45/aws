# DynamoDB — Write Sharding (Fragmentación de escritura)

Solución para hot partition cuando la PK tiene poca cardinalidad por naturaleza.

---

## El problema

Ejemplo: tabla de votos, PK = `Candidate_ID`. Solo 2 candidatos = 2 particiones = hot partition.

## La solución

Agregar un **sufijo** al valor de la PK al escribir:

- `Candidate_A` → `Candidate_A-11`, `Candidate_A-73`, `Candidate_A-42`...
- 2 candidatos × 100 sufijos = 200 PKs posibles = tráfico distribuido

## Dos métodos

| Método | Cómo | Ventaja | Desventaja |
|---|---|---|---|
| Sufijo aleatorio | random(1-100) | Simple | Para leer hay que consultar TODAS las variantes |
| Sufijo calculado | hash(voter_id) mod 100 | Podés encontrar un voto específico sin buscar en todas | Más lógica en la app |

## Tradeoff

- **Escribir** → rápido y distribuido
- **Leer/agregar** → más trabajo (query a cada variante y sumar)

## Para el examen

- "PK con poca cardinalidad y throttling" → write sharding
- "Distribuir escrituras entre particiones" → sufijo aleatorio o calculado en la PK
