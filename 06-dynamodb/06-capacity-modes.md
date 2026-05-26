# DynamoDB — Modos de Capacidad (Provisioned vs On-Demand)

Controla cómo se gestiona la capacidad de lectura/escritura de la tabla. Es una decisión de **costo + tolerancia a picos**.

---

## Modo Provisioned (Aprovisionado) — default

### Cómo funciona

- Vos especificás el número de **lecturas y escrituras por segundo** que esperás.
- Pagás por **RCU (Read Capacity Units)** y **WCU (Write Capacity Units)** *reservadas*, las uses o no.
- Podés activar **autoescalado** para que ajuste RCU/WCU según métricas de CloudWatch.

### Cuándo conviene

Cargas **predecibles y estables**:
- Apps internas con tráfico constante.
- Backends con patrones diarios conocidos (ej: más uso en horario laboral).

### Los dos peligros

1. **Sobreaprovisionar** → pedís 1000 RCU y usás 100 → pagás capacidad ociosa.
2. **Subaprovisionar** → pedís 100 RCU y necesitás 1000 → **throttling** (`ProvisionedThroughputExceededException`).

### Trampa del autoescalado

El autoescalado de Provisioned **NO es instantáneo**:
- Reacciona a métricas de CloudWatch.
- Tarda **minutos** en subir la capacidad.
- Si el pico dura 30 segundos → no llega a tiempo → **throttling igual**.

> **Conclusión:** autoescalado sirve para cambios graduales, NO para picos repentinos.

### Burst Capacity (Capacidad de ráfaga)

DynamoDB **NO es rígido** con el límite provisionado. Te guarda la capacidad **no usada** de los últimos **5 minutos (300 segundos)** como reserva de emergencia.

Si superás tu capacidad provisionada, DynamoDB tira de esa reserva acumulada antes de tirarte el error.

#### Analogía — plan de datos del celular

- Tenés **1 GB/día** contratado (capacidad provisionada).
- Si un día usás solo 200 MB → te "sobran" 800 MB.
- La compañía te los acumula para los próximos días.
- Si un día querés ver una película y usás 3 GB → primero descuenta el GB del día, después tira de la reserva.
- Si la reserva también se vacía → te corta internet (= `ProvisionedThroughputExceededException`).

#### Cómo se acumula

> Burst Capacity = capacidad **no usada** en los últimos **300 segundos**.

Ejemplo:
- Provisionaste 10 RCU.
- Usaste solo 4 RCU/s promedio en los últimos 5 minutos.
- Se acumularon **6 RCU × 300s = 1800 RCU** de ráfaga.
- Si de golpe necesitás 50 RCU/s por unos segundos → DynamoDB usa la reserva sin romperse.

#### Cuándo se rompe — el error de examen

`ProvisionedThroughputExceededException` aparece cuando:

1. Estás consumiendo más de lo provisionado, **Y**
2. La Burst Capacity acumulada ya se gastó.

Si solo se cumple (1) pero todavía hay burst → **NO hay error**, DynamoDB usa la reserva silenciosamente.

> **Trampa de examen:** el error NO aparece "apenas te pasás del límite". Aparece cuando se acaba la ráfaga.

#### Limitaciones

- **AWS no garantiza la ráfaga.** Docs oficiales: *"reserved for background maintenance, may not always be available"*.
- **No la uses como estrategia.** Es un colchón, no una solución.
- Si tu tráfico supera lo provisionado de forma **sostenida**, te quedás sin ráfaga rápido.

#### Qué hacer ante `ProvisionedThroughputExceededException`

1. Aumentar RCU/WCU manualmente.
2. Activar autoescalado (si picos son graduales).
3. Cambiar a On-Demand (si picos son abruptos).
4. Implementar **reintentos con exponential backoff** en el cliente (el SDK de AWS lo hace por default).
5. Revisar el diseño de la tabla — puede ser **hot partition**, no falta de capacidad.

---

## Modo On-Demand (Bajo Demanda)

### Cómo funciona

- DynamoDB ajusta capacidad **automáticamente y al instante** según tu carga.
- Pagás por **cada request** que hacés.
- No planificás nada, no hay throttling por capacidad mal calculada.

### Cuándo conviene

- Cargas **impredecibles** con picos repentinos pronunciados.
- Black Friday, lanzamientos virales, eventos puntuales.
- Apps nuevas donde aún no sabés el patrón de tráfico.
- Entornos de dev/test (poco uso).

### Tradeoff

- **Más caro POR request** que Provisioned.
- **Pero** no pagás por capacidad ociosa.
- Si tu tráfico es muy errático (mucho idle + picos), puede salir **más barato en total** que Provisioned sobredimensionado.

> **Regla mental:** On-Demand cobra más por request, pero te ahorra pagar por capacidad reservada que no usás.

---

## Cambio entre modos

- Podés cambiar de Provisioned a On-Demand y viceversa.
- **Limitación importante:** solo **una vez cada 24 horas**.
- Si te equivocás de modo, te bancás 24 horas hasta poder cambiar.

---

## Tabla de decisión rápida

| Situación                                            | Modo correcto             |
|------------------------------------------------------|---------------------------|
| Tráfico estable y predecible                         | **Provisioned**           |
| Patrones diarios conocidos (ej: oficina 9-18h)       | Provisioned + autoescalado |
| Black Friday, viral en Twitter, picos impredecibles  | **On-Demand**             |
| App nueva, tráfico desconocido                       | **On-Demand**             |
| Dev / test                                           | **On-Demand**             |
| Carga constante 24/7 con uso alto                    | **Provisioned**           |

---

## RCU y WCU — vista rápida

Mención breve, se profundiza en su propio apunte cuando aparezca en el curso:

- **RCU (Read Capacity Unit)** → 1 lectura *eventually consistent* de hasta **4 KB** por segundo.
  (O 0.5 lectura *strongly consistent*.)
- **WCU (Write Capacity Unit)** → 1 escritura de hasta **1 KB** por segundo.

Vas a tener que calcularlos en el examen a partir del tamaño de los items y del tipo de lectura.

---

## Trampas típicas del examen

1. **"Picos repentinos impredecibles"** → respuesta es **On-Demand**, NO "Provisioned con autoescalado".

2. **"Carga estable, queremos minimizar costos"** → **Provisioned**.

3. **"App nueva, no conocemos el tráfico"** → **On-Demand** (después medís y migrás si conviene).

4. **"La app recibe error ProvisionedThroughputExceededException"** → estás en Provisioned y subaprovisionado. Opciones:
   - Aumentar RCU/WCU manualmente.
   - Activar autoescalado (si los picos son graduales).
   - Cambiar a On-Demand (si los picos son abruptos).

5. **"Queremos cambiar de modo dos veces en un día"** → no se puede, **límite de 24 horas**.

6. **"Cuál modo es más barato?"** → trampa, depende del patrón de tráfico. Provisioned es barato POR request pero caro si sobredimensionás; On-Demand es caro POR request pero no paga capacidad ociosa.

---

## Auto-test mental

1. ¿Por qué autoescalado de Provisioned NO resuelve picos repentinos?
2. ¿En qué caso On-Demand puede salir más barato que Provisioned?
3. ¿Cuánto hay que esperar para cambiar de modo?
4. ¿Qué error ves cuando subaprovisionás en modo Provisioned?
5. ¿Cuánto es 1 RCU y 1 WCU?
6. ¿Qué es la Burst Capacity y cuánto tiempo de "no uso" acumula?
7. ¿Cuándo aparece `ProvisionedThroughputExceededException` exactamente?
8. Si superás el límite provisionado por 10 segundos pero tenés mucha ráfaga acumulada, ¿hay error?
