# EC2 Purchasing Options — cómo entenderlas (sin memorizar)

Este bloque es de examen. La clave NO es memorizar lista, sino responder 3 preguntas:

1. ¿Tu carga es **estable** o **interrumpible**?
2. ¿Buscás **descuento** o **capacidad garantizada**?
3. ¿Necesitás **flexibilidad** o podés comprometerte 1–3 años?

---

## 1) On-Demand (bajo demanda)

- Pagás por uso, sin compromiso largo.
- Precio más alto relativo, máxima simplicidad/flexibilidad.
- Ideal para: picos, pruebas, cargas impredecibles o de corto plazo.

Palabra mental: **"arranco ya"**.

---

## 2) Reserved Instances (RI)

- Compromiso de **1 o 3 años** para obtener descuento.
- Son principalmente un **beneficio de facturación** (no una instancia física “reservada” en sí).
- Clases:
  - **Standard RI**: mayor descuento, menos flexibilidad.
  - **Convertible RI**: menos descuento, más flexibilidad para cambios.

Nota de precisión:
- Los **Zonal RI** pueden incluir beneficio de capacidad en AZ; los **Regional RI** apuntan más al descuento de facturación.

Palabra mental: **"descuento por compromiso"**.

---

## 3) Savings Plans

- Compromiso de gasto por hora (USD/h) por 1 o 3 años.
- Más flexibles que RI para cambios de familia/tamaño/servicio (según plan).
- AWS recomienda normalmente Savings Plans por simplicidad y flexibilidad.

Palabra mental: **"descuento flexible"**.

---

## 4) Spot Instances

- Usan capacidad ociosa con gran descuento.
- **Interrumpibles** (AWS puede recuperarlas).
- Ideal para batch, procesamiento asíncrono, cargas tolerantes a interrupción.

Palabra mental: **"barato pero expulsable"**.

---

## 5) Capacity Reservations (On-Demand Capacity Reservations)

- Reservan **capacidad** de cómputo en una **AZ específica**.
- Se usan cuando necesitás garantía de disponibilidad de capacidad.
- No son, por sí solas, el mejor mecanismo de descuento.

Palabra mental: **"garantía de cupo"**.

---

## 6) Dedicated Instances vs Dedicated Hosts

### Dedicated Instances
- Tu instancia corre en hardware dedicado para un solo cliente (no compartido con otros clientes).
- Menos control de afinidad/licencias que Dedicated Hosts.

### Dedicated Hosts
- Reservás un servidor físico completo.
- Mayor control de colocación y casos de compliance/licenciamiento.

Palabra mental:
- Dedicated Instance = **aislamiento**.
- Dedicated Host = **aislamiento + control físico**.

---

## 7) Decision tree de examen (rápido)

1. **¿Se puede interrumpir?**
   - Sí → Spot.
   - No → seguir.

2. **¿Necesitás capacidad garantizada en una AZ?**
   - Sí → Capacity Reservation (posible combinación con ahorro).
   - No → seguir.

3. **¿Carga estable por 1–3 años?**
   - Sí → Savings Plans (o RI según caso).
   - No → On-Demand.

4. **¿Requisito fuerte de aislamiento/licencias?**
   - Sí → Dedicated Instances/Hosts.

---

## 8) Trampas de examen típicas

1. Confundir **descuento** con **capacidad garantizada**.
2. Creer que Spot sirve para cargas críticas sin tolerancia a interrupción.
3. Tratar RI como “instancia física reservada” en todos los casos.
4. Olvidar que Dedicated Host != Dedicated Instance.
5. Ignorar que Capacity Reservations pueden combinarse con mecanismos de ahorro.

---

## 9) Regla de oro

- **On-Demand** = flexibilidad inmediata
- **Savings/RI** = ahorro por compromiso
- **Spot** = máximo ahorro con riesgo de interrupción
- **Capacity Reservation** = garantía de capacidad
- **Dedicated** = aislamiento/compliance
