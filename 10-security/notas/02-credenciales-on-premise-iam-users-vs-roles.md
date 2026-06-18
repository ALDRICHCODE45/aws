# Credenciales AWS para apps on-premise

## Regla rápida según dónde corre la app

| Dónde corre | Solución |
| ----------- | -------- |
| **EC2 / Lambda / ECS** | **IAM Role** (instance profile / execution role). NUNCA access keys. |
| **On-premise / VM local** | **IAM user + access keys en `~/.aws/credentials`** |
| On-prem con "máxima seguridad" | IAM Roles Anywhere o IAM Identity Center con federación |

## Por qué on-prem NO puede "asumir un rol directamente"

Los roles los asumen identidades AWS (servicios, federadas, o users con credenciales previas). Una VM on-prem no es ninguna → necesita un IAM user con access keys primero.

## Trampa de enunciado

| Si dice | Buscar |
| ------- | ------ |
| "más apropiada" / "más simple" | Solución **mínima estándar** (IAM user + keys para on-prem) |
| "máxima seguridad" / "sin credenciales permanentes" | STS, federación, IAM Roles Anywhere |

> El examen NO siempre premia "lo más seguro del mundo real". Premia la respuesta **correcta para el caso descrito**.

## Opciones trampa típicas

- "Asignar rol IAM directamente a VM on-prem" → imposible sin IAM Roles Anywhere
- "Guardar username/password en `~/.aws/credentials`" → el SDK usa **access key/secret**, no user/pass
- "Hardcodear access keys en código" / "usar root account" → descarte automático
