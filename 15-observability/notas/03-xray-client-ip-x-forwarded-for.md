# X-Ray — IP real del cliente detrás de un ELB

## Pregunta tipo

App en EC2 detrás de un ALB, con X-Ray. ¿De dónde saca X-Ray la IP del cliente?

## Respuesta

Del header **`X-Forwarded-For`**.

## Por qué

El ALB **termina la conexión** → la "IP de origen del paquete IP" que ve la EC2
es la IP del ALB, NO la del cliente. El ALB agrega la IP real del cliente en el
header `X-Forwarded-For`. X-Ray la lee de ahí.

## Distractores

- ❌ `X-Forwarded-Host` → host original, NO la IP.
- ❌ `ipAddress` query param → inventado.
- ❌ "IP de origen del paquete IP" → esa es la del balanceador (trampa principal).

## Regla

Detrás de cualquier ELB / proxy: **IP real del cliente = `X-Forwarded-For`**.
(Acertada por suerte en simulacro #2 → ahora clavada.)

## Pregunta de prueba

Una app en EC2 detrás de un ALB usa X-Ray. ¿De dónde obtiene X-Ray la IP del cliente?

A) De la dirección IP de origen del paquete IP
B) Del header `X-Forwarded-For`
C) Del header `X-Forwarded-Host`
D) Del query param `ipAddress`

<details><summary>Respuesta</summary>

**B** (`X-Forwarded-For`): el ALB termina la conexión y agrega la IP real ahí.
Cuándo sería cada una:
- **IP de origen del paquete** → sería la IP del ALB, no la del cliente (trampa).
- **X-Forwarded-Host** → es el host original, no la IP.
- **ipAddress query param** → inventado.
</details>
