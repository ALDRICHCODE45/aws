# ECS placement — Strategies vs Constraints vs Cluster Query Language vs Task Sets

Cuatro conceptos de ECS que se confunden. Distinguir por QUÉ hace cada uno.

| Concepto | Qué es | Disparador |
| -------- | ------ | ---------- |
| **Task Placement Strategies** | ALGORITMO de cómo repartir tareas (`binpack`/`spread`/`random`) | "cómo distribuir", "reducir costo", "alta disponibilidad" |
| **Task Placement Constraints** | REGLA que LIMITA dónde va una TAREA (`distinctInstance`, `memberOf`) | "limitar/restringir dónde corre la tarea", "una por instancia" |
| **Cluster Query Language** | LENGUAJE de EXPRESIONES para AGRUPAR/seleccionar **instancias** por atributos | "definir expresiones", "agrupar instancias por atributos" |
| **Task Sets** | Conjunto de tareas para blue/green / deployment controller externo | "blue/green con external controller" |

## La trampa Constraints vs Cluster Query Language
- Los **constraints** (`memberOf`) USAN el Cluster Query Language por debajo → están relacionados.
- Pero **constraint = regla que limita una TAREA**; **Cluster Query Language = lenguaje para
  escribir las expresiones que agrupan INSTANCIAS por atributos**.
- Si la pregunta pide "definir expresiones para agrupar instancias según atributos
  (tipo de instancia, versión SO, zona, custom)" → **Cluster Query Language**, no constraint.

## Ganchos
"expresiones" + "agrupar/seleccionar instancias por atributos" → **Cluster Query Language**.
"regla que limita dónde corre la TAREA" → **Constraint**. "cómo repartir" → **Strategy**.

## Pregunta de prueba

En ECS (EC2 launch type) querés agrupar instancias de contenedor en grupos lógicos según
atributos (tipo de instancia, versión de SO, zona) definiendo **expresiones**, para que ECS
coloque tareas según las características del grupo. ¿Qué capacidad usás?

A) Conjuntos de tareas (Task Sets)
B) Estrategias de colocación de tareas (Task Placement Strategies)
C) Restricciones de colocación de tareas (Task Placement Constraints)
D) Lenguaje de consulta de clúster (Cluster Query Language)

<details><summary>Respuesta</summary>

**D** (Cluster Query Language): es el lenguaje de expresiones para agrupar/seleccionar instancias por atributos.
- **C** Constraints → regla que limita una tarea; USA cluster query pero no es "definir las expresiones de agrupamiento" (trampa relacionada).
- **B** Strategies → algoritmo de reparto (binpack/spread/random).
- **A** Task Sets → blue/green con controller externo.
</details>
