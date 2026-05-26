# DynamoDB — Modelado de Relaciones (NoSQL)

> Este apunte es DENSO. Si no se entiende a la primera, está bien. Releelo. La idea central
> está al principio (analogía de los papelitos). Lo demás son aplicaciones.

---

## La verdad incómoda

En NoSQL **NO existen las relaciones formales**:

- No hay `FOREIGN KEY`.
- No hay `JOIN`.
- No hay integridad referencial garantizada por el motor.

Si borrás un usuario, sus posts quedan huérfanos y la base **no te avisa**.

Las "relaciones" se modelan con **cómo guardás los datos**, no con reglas del motor.

---

## Analogía mental: la caja de papelitos

Olvidate de DynamoDB por un rato. Imaginá una **caja gigante** donde guardás papelitos.
Cada papelito tiene **2 etiquetas** pegadas:

- **Etiqueta AZUL** arriba → en realidad se llama **Partition Key (PK)**
- **Etiqueta NARANJA** abajo → en realidad se llama **Sort Key (SK)**

Cada papelito tiene además **contenido libre** (los atributos del item).

```
┌─────────────────────────┐
│  AZUL:    ???           │   ← Partition Key
│  NARANJA: ???           │   ← Sort Key
│  contenido: { ... }     │   ← atributos
└─────────────────────────┘
```

### La regla mágica de la caja

> "Si me pedís todos los papelitos con la **MISMA AZUL**, te los doy AL TOQUE,
> ordenaditos por NARANJA."

Eso es TODO lo que hace DynamoDB. Lo demás es cómo usás esa regla.

---

## Ejemplo concreto: users + posts

Aldrich (`u1`) tiene 2 posts. Juan (`u2`) tiene 1 post. En **una sola caja** (tabla):

```
AZUL          NARANJA       contenido
─────────────────────────────────────────────────────────────
USER#u1       PROFILE       { nombre: "Aldrich", email: "a@x.com" }
USER#u1       POST#p1       { titulo: "Hola mundo", body: "..." }
USER#u1       POST#p2       { titulo: "DynamoDB rocks", body: "..." }
USER#u2       PROFILE       { nombre: "Juan", email: "juan@gmail.com" }
USER#u2       POST#p99      { titulo: "Mi primer post", body: "..." }
```

Observá lo importante:

- Todos los papelitos de Aldrich tienen la **misma AZUL** (`USER#u1`) → viven juntos.
- La NARANJA los diferencia (`PROFILE` vs `POST#p1` vs `POST#p2`).
- Users y posts conviven en la **misma tabla**. Esto se llama **Single Table Design**.

---

## Qué le podés pedir a la caja

### Pedido 1: "Dame TODO lo de Aldrich"

```
Query: PK = "USER#u1"
```

Devuelve los **3 items juntos**, ordenados por SK:
1. `PROFILE`
2. `POST#p1`
3. `POST#p2`

→ Una sola operación, una sola partición, milisegundos. **Sin JOIN.**

### Pedido 2: "Dame SOLO los posts de Aldrich"

```
Query: PK = "USER#u1" AND SK begins_with "POST#"
```

Devuelve solo los 2 posts. El profile queda fuera.

### Pedido 3: "Dame SOLO el perfil de Aldrich"

```
Query: PK = "USER#u1" AND SK = "PROFILE"
```

Devuelve solo el perfil.

---

## El truco: prefijos en la Sort Key

`USER#`, `POST#`, `PROFILE`, `COMMENT#` son **convenciones inventadas por developers**.
DynamoDB **no las entiende** — son solo strings.

Pero a vos te sirven para:
- Diferenciar tipos de items dentro de la misma tabla.
- Hacer queries jerárquicas con `begins_with`.

Patrón típico:

```
USER#<id>           PROFILE
USER#<id>           POST#<post_id>
USER#<id>           COMMENT#<comment_id>
USER#<id>           FOLLOW#<followed_user_id>
```

---

## Las 3 estrategias para "relaciones" en DynamoDB

### Estrategia A — Tablas separadas (estilo SQL)

```
Tabla USERS
user_id (PK)    | name      | email
u1              | Aldrich   | a@x.com

Tabla POSTS
post_id (PK)    | user_id   | title
p1              | u1        | Hola mundo
```

**Problema:** filtrar posts por `user_id` requiere un `Scan` (lentísimo, carísimo),
porque `user_id` no es la PK de POSTS.

→ Mala estrategia sola. Solo viable si agregás un **GSI** (Global Secondary Index).

### Estrategia B — Single Table Design (lo que vimos arriba)

Una sola tabla, todos los tipos juntos, PK/SK bien diseñadas con prefijos.

→ El patrón canónico que AWS recomienda. **CAE EN EXAMEN.**

### Estrategia C — Desnormalización / Duplicación

En SQL te enseñaron "nunca dupliques datos". En NoSQL, **duplicar es una HERRAMIENTA**.

Ejemplo: si en cada post querés mostrar el nombre del autor sin segunda query:

```
PK          | SK          | post_title     | author_id   | author_name
USER#u1     | POST#p1     | Hola mundo     | u1          | Aldrich
USER#u1     | POST#p2     | DynamoDB rocks | u1          | Aldrich
```

Sí, `Aldrich` está duplicado. **Y está bien.**

**Por qué:**
- Storage es BARATO.
- Lecturas son CARAS (latencia + RCU).
- Si Aldrich cambia el nombre → actualizás los N posts (async vía Streams + Lambda).

**Tradeoff fundamental:**
| SQL                              | NoSQL                            |
|----------------------------------|----------------------------------|
| Optimiza ESCRITURAS              | Optimiza LECTURAS                |
| Una fila por entidad, sin duplicar | Duplicar es OK para evitar JOINs |

---

## El caso de los comentarios — depende de la query

Si en el post `p1` de Aldrich, Juan (`u2`) deja un comentario `c1`:

### Opción A — el comentario vive con el post

```
PK: POST#p1
SK: COMMENT#c1
contenido: { autor: "u2", texto: "Buenísimo!" }
```

→ Sirve para: "Dame el post `p1` con todos sus comentarios".

### Opción B — el comentario vive con el autor

```
PK: USER#u2
SK: COMMENT#c1
contenido: { post: "p1", texto: "Buenísimo!" }
```

→ Sirve para: "Dame todos los comentarios que hizo Juan".

### Opción C — duplicar (las dos)

Si necesitás ambas queries, **guardás dos copias** del comentario.

**Conclusión:** ninguna opción es "la correcta" en abstracto.
La respuesta DEPENDE de las queries que tu app vaya a hacer.

---

## La filosofía clave (frase de examen)

> **En SQL, primero modelás los datos. Después ves cómo consultarlos.**
> **En DynamoDB, primero modelás las queries. Después ves cómo guardar los datos.**

Esto se llama **"design for access patterns"**.

Si no sabés cómo vas a consultar los datos, **no podés diseñar tu tabla DynamoDB**. Punto.

---

## Tabla de decisión rápida

| Pregunta                                                   | Estrategia              |
|------------------------------------------------------------|-------------------------|
| Entidades que se consultan siempre juntas                  | Single Table Design     |
| Entidades totalmente independientes entre sí               | Tablas separadas (raro) |
| Necesito datos de A cuando consulto B                      | Desnormalizar (duplicar) |
| Quiero hacer un JOIN                                       | DETENETE. Replanteá.    |

---

## Trampas típicas del examen

1. **"Cómo modelar relación 1:N en DynamoDB?"**
   → Partition Key compartida + Sort Key con prefijos (Single Table).

2. **"Cómo evitar JOINs?"**
   → Desnormalización + Single Table Design.

3. **"Por qué duplicar datos en DynamoDB?"**
   → Para minimizar lecturas, storage es más barato que latencia.

4. **"Cómo mantener datos duplicados consistentes?"**
   → DynamoDB Streams + Lambda (replicación async).

5. **Pregunta con palabras clave "design for access patterns"**
   → Respuesta correcta casi siempre incluye Single Table o GSI.

---

## Próximos pasos (cuando avances en el curso)

- **GSI (Global Secondary Index)** → permite "consultar al revés" sin Scan.
- **LSI (Local Secondary Index)** → otro ángulo de ordenamiento dentro de la misma partición.
- **Query vs Scan** → entender bien por qué Scan es el enemigo.

Con esos tres temas, vas a poder modelar CUALQUIER relación en DynamoDB.

---

## Auto-test mental antes de cerrar el apunte

Respondete sin mirar el contenido:

1. ¿Qué significa "AZUL" y "NARANJA" en el mundo real de DynamoDB?
2. ¿Por qué guardamos users y posts en la MISMA tabla?
3. Si quiero "todos los comentarios de un user", ¿qué AZUL y NARANJA uso?
4. ¿Por qué duplicar el nombre del autor en cada post NO está mal?
5. ¿Qué frase resume la filosofía de modelado en DynamoDB?

Si dudás en alguna, releé la sección correspondiente.
