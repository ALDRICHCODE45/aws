# KMS — Key Management (tipos, rotación, policy vs grants)

## Tipos de key

| Tipo                             | Control                         | Costo  |
| -------------------------------- | ------------------------------- | ------ |
| **AWS owned**                    | invisible, ni la ves            | gratis |
| **AWS managed** (`aws/servicio`) | AWS rota, no controlás política | gratis |
| **Customer managed (CMK)**       | VOS: política, rotación, grants | pago   |

→ "control / auditoría / rotación propia" = **Customer managed**.

## Rotación

- Customer managed con rotación automática: **cada 1 año** (configurable).
- Material viejo se guarda para descifrar datos viejos.
- Rotar **NO cambia el key ID/ARN** → la app no se entera. Trampa: "¿rotar rompe
  referencias?" → NO.
- SSE-C y material importado: NO hay rotación automática.

## Permisos: Key Policy vs Grants vs IAM

- **Key Policy**: control PRINCIPAL, obligatoria, vive en la key.
- **Grants**: permisos **temporales, programáticos, revocables** → "acceso
  temporal/delegado a la key".
- **IAM policy**: complementa, pero la key policy manda.

### Trampa cross-account

Acceso a una KMS key desde otra cuenta → necesitás **key policy que lo permita +
IAM en la otra cuenta**. Solo IAM NO alcanza.

## Pregunta de prueba

Una app financiera exige control total de la clave de cifrado: política propia,
rotación gestionada por vos y auditoría de uso. ¿Qué tipo de KMS key usás?

A) AWS owned key
B) AWS managed key (`aws/s3`, etc.)
C) Customer managed key (CMK)
D) Una data key

<details><summary>Respuesta</summary>

**C** (Customer managed / CMK): vos controlás política, rotación y grants.
Cuándo sería cada una:
- **AWS owned** → ni la ves, cero control.
- **AWS managed** → AWS rota y controla la política; no tenés control fino.
- **data key** → es la clave que cifra los datos (envelope), no una key de KMS para gestionar.
</details>
