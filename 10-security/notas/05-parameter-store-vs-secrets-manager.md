# Parameter Store vs Secrets Manager vs CloudHSM

## Tabla comparativa

| | Parameter Store SecureString | Secrets Manager | CloudHSM |
| -- | --------------------------- | --------------- | -------- |
| Cifrado KMS | ✅ | ✅ | HSM dedicado |
| **Rotación automática** | ❌ Manual | ✅ Auto | N/A |
| Integración RDS/Aurora/Redshift | ❌ | ✅ nativa | ❌ |
| Generar password aleatoria | ❌ | ✅ | ❌ |
| Costo | **Gratis** (estándar) | $0.40/mes/secret | **$$$** |

## Disparadores

| Pregunta menciona | Servicio |
| ----------------- | -------- |
| "almacenar config / connection string / API key" + "cifrado" | **Parameter Store SecureString** |
| "**rotación automática**" / "credenciales DB que rotan" | **Secrets Manager** |
| "generar password aleatoria" | **Secrets Manager** |
| "compliance FIPS 140-2 Level 3" / "HSM dedicado" | **CloudHSM** |

## Trampas

- **CloudHSM** suena "más seguro" → es trampa para casi cualquier caso normal. Es overkill caro.
- **Variables de entorno cifradas + copiar a cada Lambda** → anti-pattern (no es compartido).
- **IAM Role** → resuelve permisos, NO almacenamiento de secretos.

## Regla rápida

Si no menciona rotación → **Parameter Store** (gratis).
Si menciona rotación o RDS → **Secrets Manager**.
Si menciona HSM/FIPS → **CloudHSM**.
