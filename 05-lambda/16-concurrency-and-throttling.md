# Lambda — Concurrencia y throttling

Resumen corto para examen.

---

## Concepto base

**Concurrencia** = invocaciones de Lambda corriendo AL MISMO TIEMPO (en vuelo).
No es lo mismo que cantidad total de invocaciones.

- 1000 invocaciones de 100 ms → concurrencia baja.
- 100 invocaciones de 10 s → concurrencia alta.

---

## Límite de cuenta

- **1000 ejecuciones concurrentes por región** por defecto.
- **COMPARTIDO** entre TODAS las Lambdas de la cuenta en esa región.
- Se puede subir abriendo ticket de soporte.

---

## Reserved Concurrency (concurrencia reservada)

Reservás N slots para UNA Lambda específica. Hace dos cosas:

- **Piso garantizado**: nadie se los puede robar.
- **Techo máximo**: esa Lambda no puede pasar de N.

Sirve para evitar el **noisy neighbor** (una Lambda glotona que se come los 1000 slots y deja a las demás sin ejecutarse).

---

## Throttling (estrangulamiento)

Lo que pasa cuando llega una invocación y NO hay slots libres.

| Tipo de invocación | Comportamiento |
|---|---|
| **Síncrona** | Devuelve **429 ThrottleError** al caller |
| **Asíncrona** | Reintenta automático + backoff exponencial → **Dead Letter Queue (DLQ)** |

---

## Async + throttling: reintentos

Cuando una invocación asíncrona se topa con throttling (o errores 5xx):

1. Lambda devuelve el evento a una cola interna.
2. Reintenta con **backoff exponencial**: 1 s, 2 s, 4 s... hasta máx 5 min entre intentos.
3. Sigue reintentando hasta **6 horas**.
4. Si sigue fallando → **DLQ**.

Síncronas NO tienen reintento automático. El caller decide qué hacer.

---

## Cold start

- Primera invocación o contenedor nuevo.
- AWS descarga código, inicializa runtime, ejecuta el código fuera del handler.
- La primera petición tiene más latencia.
- Init pesado (SDKs, conexiones DB, modelos ML) = cold start largo.
- En 2019 AWS mejoró drásticamente cold start en VPC.

---

## Provisioned Concurrency (concurrencia aprovisionada)

Le decís a AWS: "tené N contenedores SIEMPRE calientes, ya inicializados".

- Las invocaciones que entran NO sufren cold start mientras quepan en los N.
- Cuesta más (pagás por mantenerlos vivos).
- Se puede escalar con **Application Auto Scaling** (por horario o uso).

---

## Reserved vs Provisioned (la gran comparativa)

| Concepto | Para qué sirve |
|---|---|
| **Reserved** | Limitar/garantizar SLOTS concurrentes |
| **Provisioned** | Eliminar COLD STARTS |

No son lo mismo. Se pueden combinar:

```
my-function-PROD:
├── Reserved: 300       ← techo de concurrencia
└── Provisioned: 200    ← 200 contenedores siempre calientes
```

- Hasta 200 concurrentes → sin cold start.
- Entre 200 y 300 → cold start en los nuevos.
- Más de 300 → throttling.

---

## Ejemplo realista

Tres Lambdas en una cuenta (límite total 1000):

- `checkout` (crítica, sin cold start): reserved = 400, provisioned = 100.
- `notify` (asíncrona, menos crítica): reserved = 200.
- `cleanup` (job interno, asíncrona): sin reservar, usa los 400 libres.

Si `checkout` explota en tráfico, no se come a `notify` ni a `cleanup`.

---

## Puntos clave para examen

- Límite default: **1000 concurrentes por región** (cuenta).
- **Reserved** = piso + techo a la vez.
- **Provisioned** = mata cold starts pagando.
- **Síncrona + throttling** = 429 ThrottleError.
- **Asíncrona + throttling** = reintentos con backoff hasta 6 h → DLQ.
- Sin reserved en Lambdas activas → **noisy neighbor problem**.
- Cold start en VPC mejorado desde 2019.
