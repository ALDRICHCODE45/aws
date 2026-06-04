# Burst Capacity y Respuesta ante Throttling

## Burst Capacity

- Acumula capacidad no usada de los últimos **300 segundos**
- AWS **NO la garantiza** — es mejor esfuerzo
- Si se agota → `ProvisionedThroughputExceededException`

## Cadena de respuesta ante throttling

1. **Exponential backoff** → SDKs de AWS lo hacen automáticamente
2. **Auto Scaling** → tarda minutos, no sirve para picos repentinos
3. **On-Demand** → si picos son impredecibles y recurrentes (2.5x más caro, cambio cada 24h)
4. **Causa raíz** → Hot Key: DAX | Hot Partition: write sharding | Ítems grandes: S3

## Tabla rápida

| Situación                 | Respuesta           |
| ------------------------- | ------------------- |
| Pico corto, temporal      | Burst lo absorbe    |
| SDK reintenta             | Exponential backoff |
| Picos predecibles         | Auto Scaling        |
| Picos impredecibles       | On-Demand           |
| 1 PK con 80% tráfico      | DAX                 |
| Mala distribución general | Write sharding      |
