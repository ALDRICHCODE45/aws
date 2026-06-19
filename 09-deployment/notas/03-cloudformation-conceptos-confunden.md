# CloudFormation — 4 conceptos que confunden

| Concepto            | Tiempo | Para qué                              |
| ------------------- | ------ | ------------------------------------- |
| **Change Set**      | FUTURO | Preview de cambios antes de aplicar   |
| **Direct Update**   | AHORA  | Aplica cambios directo (sin red)      |
| **Drift Detection** | PASADO | Detecta cambios manuales ya hechos    |
| **StackSet**        | N/A    | Replica stack en N cuentas + regiones |

## Disparadores

- "preview / antes de aplicar / qué va a cambiar" → **Change Set**
- "cambios fuera de CFN / modificación manual / compliance" → **Drift Detection**
- "múltiples cuentas / múltiples regiones / Organizations" → **StackSet**

## Trampa

**Stack ≠ StackSet**. Stack = una cuenta + una región. StackSet = multi-cuenta + multi-región.

Otros pares trampa por nombre: User Pool vs Identity Pool, Standard vs Standard-IA, EBS Snapshot vs AMI.

## Detalles que caen

- Change Set: acciones son **Add / Modify / Remove / Replace** (Replace = destroy+create, ojo downtime).
- Drift Detection: solo **reporta**, no arregla.
