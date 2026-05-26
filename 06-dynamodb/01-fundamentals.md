# DynamoDB — Fundamentos

## Qué es

Base de datos **NoSQL serverless** totalmente gestionada por AWS. No es relacional, no usa SQL, no hay servidores que administrar.

## Características clave

- **Totalmente gestionada** — AWS se encarga de hardware, parches, replicación, failover.
- **Alta disponibilidad** — replicación automática en **múltiples AZ** dentro de una región.
- **NoSQL** — no usa JOINs, no hay esquema rígido por fila (cada item puede tener atributos distintos).
- **Distribuida y escalable horizontalmente** — escala a cargas masivas sin intervención.
- **Escala soportada** — millones de peticiones por segundo, billones de filas, cientos de TB.
- **Rendimiento rápido y constante** — latencia de un solo dígito en milisegundos.
- **Integrada con IAM** — control fino de seguridad, autorización y administración.
- **Programación basada en eventos** — vía **DynamoDB Streams** (capturan cambios en items).
- **Autoescalado** — ajusta capacidad automáticamente según demanda.
- **Clases de tabla** — Standard y Standard-IA (ver `02-table-classes.md`).

## Coste — OJO con esto (importante para examen)

DynamoDB **NO es barato por default**. Puede ser muy barato o muy caro según cómo lo configures:

- Modo de capacidad (On-Demand vs Provisioned).
- Cantidad de índices secundarios.
- Tipo de lectura (eventually consistent vs strongly consistent).
- Clase de tabla (Standard vs IA).
- Volumen de Streams, backups, global tables, etc.

> Regla mental: "Es barato si lo usás bien."

## Cuándo usarla

- Aplicaciones que necesitan **baja latencia constante** a escala.
- Cargas con **patrones de acceso conocidos** (no consultas ad-hoc complejas).
- Workloads serverless (encaja perfecto con Lambda + API Gateway).
- Datos clave-valor o documentos (no para reportes analíticos complejos).

## Cuándo NO usarla

- Cuando necesitás JOINs complejos o consultas ad-hoc → usá RDS o Aurora.
- Cuando no conocés los patrones de acceso de antemano (DynamoDB se diseña ALREDEDOR de las queries).
- Análisis OLAP, reporting pesado → usá Redshift o Athena.
