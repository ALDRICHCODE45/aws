# Lab 01 — Global Infrastructure Thinking

> Este primer lab es de **diseño** (arquitectura), no de código todavía.

## Objetivo

Diseñar una base mínima para una app web considerando región, AZ y edge.

## Enunciado

Tenés una app con:

- Usuarios: 70% Latam, 30% Europa
- Frontend estático
- API backend
- Base de datos relacional
- Requisito: tolerar caída de 1 AZ

## Tu tarea

Definí en un documento:

1. Región primaria y por qué.
2. Cómo distribuís backend y DB en AZ.
3. Dónde usás edge locations.
4. Qué quedaría pendiente para tolerar caída total de región.

## Criterio de corrección

- Decisión de región justificada por latencia + negocio.
- Separación de recursos en 2+ AZ.
- Uso correcto de CloudFront/edge para estáticos.
- Diferenciación clara entre HA regional y DR multi-región.

## Entregable

Creá: `02-labs/submissions/lab-01-answer.md`
