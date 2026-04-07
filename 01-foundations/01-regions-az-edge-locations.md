# Regiones, Data Centers, AZ y Edge Locations

## 1) El problema que AWS resuelve

Si toda tu app vive en un solo edificio y ese edificio falla, tu negocio cae.

AWS separa físicamente su infraestructura para darte:

- **Baja latencia** (estar cerca del usuario)
- **Alta disponibilidad** (tolerar fallas)
- **Resiliencia ante desastres**

Pensalo como construir una ciudad:

- Región = ciudad
- AZ = barrio independiente
- Data Center = edificio
- Edge Location = mini sucursal cerca del cliente

---

## 2) Definiciones clave

## Región (Region)

Área geográfica independiente (ej: `us-east-1`, `sa-east-1`).

- Cada región tiene múltiples AZ.
- Los servicios pueden variar entre regiones.
- Elegís región por latencia, compliance, costo y disponibilidad de servicios.

## Data Center

Instalación física con servidores, red y energía.

- No suele ser una unidad de diseño para examen.
- AWS no siempre publica ubicación exacta por seguridad.

## Availability Zone (AZ)

Uno o más data centers separados físicamente dentro de una región.

- Conectadas entre sí con red de baja latencia.
- Diseñadas para aislar fallas eléctricas/red/incendio.
- Ejemplo: `us-east-1a`, `us-east-1b`, etc.

## Edge Location

Punto de presencia global para entregar contenido cerca del usuario final.

- Se usa con CloudFront (CDN), Route 53, y otros servicios edge.
- No es para desplegar infraestructura general como si fuera una región.

---

## 3) Diferencias que confunden en examen

1. **Multi-AZ** ≠ **Multi-Region**
   - Multi-AZ: alta disponibilidad dentro de una región.
   - Multi-Region: resiliencia geográfica y menor latencia global.

2. **Edge Location** ≠ **AZ**
   - AZ aloja workloads (EC2, DB, etc).
   - Edge location distribuye/cacha contenido y acelera requests.

3. Más regiones no siempre es mejor
   - Aumenta complejidad operativa, replicación y costo.

---

## 4) Criterios para elegir región (checklist real)

- **Latencia** hacia tus usuarios principales.
- **Compliance/legal** (dónde puede residir la data).
- **Servicios disponibles** en esa región.
- **Costos** (varían por región).
- **Estrategia DR** (si necesitás región secundaria).

---

## 5) Mini escenario de arquitectura

Tenés una API para usuarios de Argentina y Brasil:

- Backend principal en `sa-east-1`
- Base de datos en Multi-AZ
- CloudFront para contenido estático global

Resultado:

- Respuesta rápida localmente
- Alta disponibilidad regional
- Mejor performance para assets globales

---

## 6) Checkpoint rápido (sin mirar)

1. ¿Cuál es la diferencia entre Multi-AZ y Multi-Region?
2. ¿Por qué una edge location no reemplaza una región?
3. Nombrá 4 criterios para elegir región.
4. Si tu app debe seguir viva ante caída total de una región, ¿qué necesitás?
