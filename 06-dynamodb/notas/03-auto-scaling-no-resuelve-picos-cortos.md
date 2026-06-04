# Auto Scaling NO resuelve picos cortos

## Cadena de defensa ante un pico

1. **Burst Capacity** → absorbe picos cortos (usa capacidad no usada de últimos 300s)
2. **Auto Scaling** → tarda minutos, llega tarde para picos de segundos
3. **On-Demand** → solución si los picos son impredecibles Y recurrentes

## Regla para el examen

| Pico                         | Respuesta                 |
| ---------------------------- | ------------------------- |
| Corto, temporal, infrecuente | Burst Capacity lo absorbe |
| Impredecible y recurrente    | On-Demand                 |
