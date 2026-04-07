# Exam Traps — Global Infrastructure

## Trampa 1
**Pregunta:** “Necesitás tolerancia a falla de un data center en una región.”

✅ Correcto: **Multi-AZ**

❌ Incorrecto: Multi-Region (sobredimensionado para ese requerimiento)

---

## Trampa 2
**Pregunta:** “Necesitás servir contenido estático más rápido globalmente.”

✅ Correcto: **CloudFront + Edge Locations**

❌ Incorrecto: desplegar toda la app en más regiones sin necesidad

---

## Trampa 3
**Pregunta:** “La empresa tiene requisito legal de residencia de datos.”

✅ Correcto: elegir región que cumpla compliance

❌ Incorrecto: priorizar solo latencia/costo

---

## Trampa 4
**Pregunta:** “Querés minimizar latencia para usuarios globales.”

✅ Correcto: combinación de regiones estratégicas + edge delivery

❌ Incorrecto: una sola región muy lejana a todos
