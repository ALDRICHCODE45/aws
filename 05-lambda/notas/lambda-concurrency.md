# Lambda — Concurrencia y límites

## Fórmula

```
Concurrencia = Req/s × Duración (seg)
```

Ej: 40 req/s × 120 seg = **4.800 concurrentes**.

## Límite default regional

**1.000 concurrent executions por región** (todas las funciones juntas).

Si tu app necesita más → **pedir Service Quota Increase a AWS**.

## Los 3 tipos de concurrencia

| Tipo | Qué hace | Cuándo |
| ---- | -------- | ------ |
| **Unreserved (default)** | Pool compartido del límite regional | Funciones normales |
| **Reserved Concurrency** | Reserva + LIMITA a N | Proteger crítica / limitar consumo |
| **Provisioned Concurrency** | Pre-warmá N instancias | Eliminar cold starts |

## ⚠️ Reserved Concurrency es arma de doble filo

- **Reserva** N (garantizado) ✅
- **LIMITA** a N (no puede pasarse) ⚠️

Si necesitás 4.800 pero el límite regional es 1.000, asignar Reserved no te salva: el techo regional sigue siendo 1.000.

## Disparadores

| Pregunta dice | Acción |
| ------------- | ------ |
| "escalar más allá de 1.000" / "evitar throttling por volumen" | **Quota Increase** |
| "proteger función crítica del consumo de otras" | **Reserved** |
| "limitar consumo de una función" | **Reserved** |
| "eliminar cold starts" / "latencia crítica" | **Provisioned** |
| "manejar picos repentinos" sin cold start | **Provisioned** |

## Trampa común

"Lambda escala automáticamente sin límites" → **FALSO**. El default es 1.000 concurrent por región.
