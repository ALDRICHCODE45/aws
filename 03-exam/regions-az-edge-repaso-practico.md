# Repaso práctico — Regiones, AZ, Data Centers y Edge Locations

Este resumen junta lo que ya dominás + ejemplos prácticos basados en preguntas tipo examen.

---

## 1) Definiciones de examen (precisas y cortas)

- **Region**: conjunto de Availability Zones (AZ) en una ubicación geográfica.
- **Availability Zone (AZ)**: una o más instalaciones físicas (data centers) discretas, con energía/red/conectividad redundante, aisladas de otras AZ.
- **Data Center**: instalación física donde corre infraestructura (servidores, red, energía).
- **Edge Location (PoP)**: punto de presencia de la red edge de AWS para acercar contenido/servicios al usuario final y reducir latencia.

---

## 2) Lo que NO hay que confundir

1. **Multi-AZ ≠ Multi-Region**
   - Multi-AZ: alta disponibilidad dentro de la misma región.
   - Multi-Region: resiliencia geográfica + mejor cobertura global.

2. **Edge Location ≠ AZ**
   - AZ aloja workloads (EC2, RDS, etc.).
   - Edge location acelera entrega/ruteo para usuarios finales.

3. **Cantidad de AZ por región**
   - No memorizar un número fijo universal para examen.
   - Puede variar por región y cambiar en el tiempo.

---

## 3) Ejemplos prácticos (basados en el mini-quiz)

### Ejemplo A — Alta disponibilidad regional

**Situación**: tu app está en una sola AZ de `us-east-1` y querés tolerar caída de un data center.

**Decisión correcta**: desplegar en múltiples AZ dentro de la misma región.

**Por qué**:

- Si cae una AZ, la otra mantiene el servicio.
- Es la estrategia clásica de HA regional.

---

### Ejemplo B — Entrega global de contenido

**Situación**: contenido estático en S3 (región lejana para parte de tus usuarios).

**Decisión correcta**: usar CloudFront con Edge Locations.

**Por qué**:

- El contenido se entrega desde puntos cercanos al usuario.
- Baja latencia y mejora experiencia global.

---

### Ejemplo C — Definición de arquitectura correcta

**Pregunta típica**: “¿Qué es una Region?”

**Respuesta fuerte de examen**:

- “Conjunto de AZs en una ubicación geográfica”

**Error común**:

- Decir “un data center grande” o mezclarlo con PoPs.

---

## 4) Checklist rápido antes de responder una pregunta

- ¿Me están pidiendo **alta disponibilidad regional**? → pensá Multi-AZ.
- ¿Me están pidiendo **resiliencia geográfica / latencia global**? → evaluá Multi-Region + edge.
- ¿Hablan de **usuarios finales y performance global**? → CloudFront/Edge Locations.
- ¿La opción tiene un número rígido de AZ “siempre”? → sospechá trampa.

---

## 5) Autoevaluación (30 segundos)

1. Definí Region, AZ y Edge Location en una línea cada una.
2. ¿Cuándo elegís Multi-AZ y cuándo Multi-Region?
3. ¿Por qué Edge Location no reemplaza una AZ?

Si podés responder esto sin mirar, ya tenés base sólida para este bloque.
