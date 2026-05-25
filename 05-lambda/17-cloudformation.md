# Lambda — Despliegue con CloudFormation

Resumen corto para examen.

---

## Dos formas de declarar una Lambda en CloudFormation

### 1. Inline (`Code.ZipFile`)

Código embebido directo en la plantilla YAML.

```yaml
Resources:
  primer:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.x
      Role: arn:aws:iam::123456789012:role/lambda-role
      Handler: index.handler
      Code:
        ZipFile: |
          import os
          def handler(event, context):
              return "hola"
```

- Solo funciones MUY simples.
- **No se pueden incluir dependencias externas**.
- Pensado para snippets cortos.

### 2. Desde S3 (`Code.S3Bucket` + `S3Key`)

Forma estándar para código real con dependencias.

```yaml
Resources:
  Function:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.handler
      Role: arn:aws:iam::123456789012:role/lambda-role
      Code:
        S3Bucket: my-bucket
        S3Key: function.zip
        S3ObjectVersion: String   # opcional, si el bucket está versionado
      Runtime: nodejs12.x
```

Propiedades clave:

- **S3Bucket**: nombre del bucket.
- **S3Key**: ruta completa al ZIP.
- **S3ObjectVersion**: versión del objeto si el bucket es versionado.

---

## Trampa importante (cae en examen)

**Si actualizás el ZIP en S3 pero NO cambiás `S3Bucket`, `S3Key` o `S3ObjectVersion`, CloudFormation NO actualiza la Lambda.**

Razón: CloudFormation detecta cambios mirando la plantilla, no S3. Si la plantilla no cambió, no hay update.

Soluciones:

1. **Bucket versionado** + referenciar `S3ObjectVersion`. Cada upload genera versión nueva.
2. **Renombrar el ZIP** cada deploy: `function-v1.zip`, `function-v2.zip`.
3. **Hash en el nombre**: `function-<hash-del-contenido>.zip`.

---

## Despliegue multi-cuenta vía S3

Patrón típico en empresas con cuentas separadas (dev/staging/prod):

- **Cuenta 1**: bucket S3 con el ZIP del código Lambda (repositorio central).
- **Cuenta 2, 3, ...**: cada una corre su CloudFormation que despliega su Lambda usando ese ZIP compartido.

```
Cuenta 1 (artefactos)        Cuenta 2 (dev)
┌──────────────────┐         ┌──────────────────┐
│  S3 Bucket       │ ←─────  │  CloudFormation  │
│  └── code.zip    │         │  └── Lambda      │
│  Bucket Policy   │         │     + Rol IAM    │
│  Allow: [2,3]    │         └──────────────────┘
└──────────────────┘
         ▲                   Cuenta 3 (prod)
         │                   ┌──────────────────┐
         └────────────────── │  CloudFormation  │
                             │  └── Lambda      │
                             │     + Rol IAM    │
                             └──────────────────┘
```

### Permisos cruzados necesarios

**Cross-account requiere los DOS lados:**

1. **Bucket Policy en Cuenta 1**: permite a Cuenta 2 y 3 leer el bucket.

```yaml
{
  "Effect": "Allow",
  "Principal": { "AWS": ["arn:aws:iam::CUENTA2:root", "arn:aws:iam::CUENTA3:root"] },
  "Action": ["s3:GetObject", "s3:ListBucket"],
  "Resource": "arn:aws:s3:::bucket-cuenta1/*"
}
```

2. **Rol IAM en Cuenta 2 y 3**: cada Lambda necesita rol con permiso de leer ese bucket.

```yaml
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:ListBucket"],
  "Resource": "arn:aws:s3:::bucket-cuenta1/*"
}
```

Si falta uno de los dos → denegado.

---

## Por qué usar este patrón

- Una única **fuente de verdad** del código.
- CI/CD sube a un solo bucket y promueve por ambiente.
- Aislamiento entre cuentas (seguridad, facturación, blast radius).

---

## Comparativa rápida

| Forma | Cuándo usar | Limitaciones |
|---|---|---|
| **Inline** | Snippets simples, una función | Sin dependencias |
| **S3 (misma cuenta)** | Caso general | Versionar para que CF detecte cambios |
| **S3 multi-cuenta** | Empresas con dev/staging/prod | Necesita bucket policy + rol IAM |

---

## Puntos clave para examen

- Inline usa `Code.ZipFile`, sin dependencias.
- S3 estándar usa `Code.S3Bucket` + `S3Key` + opcional `S3ObjectVersion`.
- Actualizar el ZIP sin cambiar la plantilla **NO** actualiza la Lambda → versionar.
- Cross-account: **bucket policy en cuenta dueña del bucket** + **IAM role en cuenta dueña de la Lambda**.
- Patrón de cuenta compartida = single source of truth para artefactos.
