# Desacoplar RDS de Elastic Beanstalk (sin perder datos)

## Problema

EB con RDS ACOPLADA al entorno → al terminar el entorno, se borra la RDS.
Querés separarla para hacer blue/green y para producción.

## Proceso correcto (doc oficial AWS)

1. **Snapshot** de la RDS.
2. **Habilitar deletion protection** en la RDS → sobrevive a la terminación del entorno.
3. Crear **nuevo entorno EB** (blue/green) con la info de conexión a la RDS existente.
4. **Eliminar la regla del Security Group que genera la dependencia.** ← CLAVE
5. Terminar el entorno viejo.

## El detalle que define la pregunta: el Security Group

- Cuando EB crea la RDS acoplada, crea una **regla de SG** que une entorno ↔ RDS.
- **ESA regla ES el acoplamiento** (a nivel de red).
- El SG de la RDS **referencia** al SG del entorno EB.
- Si NO borrás esa regla antes de terminar el entorno → AWS no puede borrar el SG
  del entorno (la RDS lo referencia) → **la terminación FALLA**.

## Estrategia de examen

- Es **blue/green**, NO canary (canary NO existe nativo en Beanstalk).
- Dos opciones casi idénticas que difieren en UN paso → **ese paso ES la pregunta**.
  Preguntá: "¿por qué agregan este paso? ¿qué pasa si lo omito?"

## Error propio (patrón #3)

Descartó la correcta porque asumió "los SG no tienen que ver con la base acoplada".
FALSO: el SG es el mecanismo de acoplamiento. No descartar por corazonada;
descartar solo cuando sabés EXACTAMENTE por qué.

## Pregunta de prueba

Vas a desacoplar una RDS acoplada a un entorno Beanstalk sin perder datos. Ya
hiciste snapshot, habilitaste deletion protection y creaste el nuevo entorno.
¿Qué paso es imprescindible ANTES de terminar el entorno viejo?

A) Cambiar la estrategia a canary
B) Eliminar la regla del Security Group que genera la dependencia
C) Borrar el snapshot para liberar espacio
D) Deshabilitar la deletion protection de la RDS

<details><summary>Respuesta</summary>

**B**. El SG de la RDS referencia al SG del entorno; si no borrás esa regla, AWS
no puede borrar el SG del entorno y la **terminación falla**.
Cuándo sería cada una:

- **canary** → nunca: Beanstalk no soporta canary nativo (es blue/green).
- **borrar snapshot / quitar deletion protection** → exactamente lo contrario:
perderías el respaldo y la protección que mantienen viva la RDS.
</details>
