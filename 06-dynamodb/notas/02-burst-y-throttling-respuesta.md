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

## Hot partition con "menor costo" → DOS capas

El throttling por hot partition se ataca en dos capas COMPLEMENTARIAS, ambas a costo CERO (solo código):

1. **Causa raíz** → **rediseñar esquema de claves / write sharding** (distribuye sobre más particiones).
2. **Manejo del síntoma** → **exponential backoff** (reintenta el `ProvisionedThroughputExceededException` con gracia).

Recordá: cada partición tiene límite DURO propio (**3000 RCU / 1000 WCU**). El total de CloudWatch
puede estar OK y aún así una sola partición se satura → errores intermitentes.

### Trampas de "menor impacto económico"
- **Aumentar capacidad provisionada** → cuesta plata y NO redistribuye la clave caliente. MAL.
- **ElastiCache delante de DynamoDB** → infra nueva (cuesta) + solo ayuda LECTURAS, no escrituras. MAL
  si el escenario menciona errores de lectura **y** escritura.

## Pregunta de prueba

Tabla DynamoDB provisionada con hot partition (CloudWatch dice que el total NO se excede,
pero hay errores intermitentes de lectura y escritura). ¿Qué DOS acciones con **menor costo**?

A) Aumentar la capacidad provisionada de lectura y escritura
B) Aplicar exponential backoff en el código de la app
C) Incorporar ElastiCache como caché delante de DynamoDB
D) Rediseñar el esquema de claves para distribuir sobre múltiples particiones

<details><summary>Respuesta</summary>

**B y D** (ambas a costo cero, atacan síntoma + causa raíz).
- **A** → cuesta plata y no redistribuye la clave caliente; el total ya estaba OK.
- **C** → infra nueva (cuesta) y solo ayuda lecturas; el escenario tiene errores de escritura también.
</details>
