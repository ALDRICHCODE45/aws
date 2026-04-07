# IAM básico (ordenado + corregido)

Este apunte consolida tus ideas de IAM con precisión de certificación.

---

## 1) Modelo mental de IAM

- **IAM** = Identity and Access Management.
- Es un servicio **global** dentro de la cuenta AWS.
- Controla:
  - **Autenticación** (quién sos)
  - **Autorización** (qué podés hacer)

---

## 2) Root user y buenas prácticas

- Toda cuenta AWS nace con un **root user** (acceso total).
- Root **no** se usa para tareas diarias.
- Recomendación:
  - habilitar MFA en root,
  - no compartir credenciales,
  - usar root solo para tareas excepcionales.

---

## 3) Usuarios, grupos y “herencia” de permisos

### Reglas clave

- Un **IAM user** puede:
  - no pertenecer a ningún grupo,
  - pertenecer a uno o varios grupos,
  - tener políticas adjuntas directamente.

- Un **IAM group**:
  - contiene usuarios,
  - **no** puede contener otros grupos.

### Caso que planteaste (admin/developers/devops)

Si un usuario pertenece a varios grupos, sus permisos efectivos son la combinación de las políticas permitidas por esos grupos + políticas directas del usuario (si existen), respetando siempre los `Deny` explícitos.

Ejemplo:
- Usuario A en `admin` + `devops`
- Usuario B en `developers` + `devops`

Ambos comparten permisos de `devops`, pero cada uno además hereda lo propio de su otro grupo.

---

## 4) Políticas IAM (JSON) — estructura correcta

### Elementos comunes

- `Version`
- `Statement` (array de declaraciones)

Dentro de cada statement, normalmente:
- `Sid` (opcional)
- `Effect` (`Allow` o `Deny`)
- `Action` o `NotAction`
- `Resource` o `NotResource`
- `Condition` (opcional)

### Ajuste IMPORTANTE de examen

- `Principal` **NO** se usa en políticas identity-based (las adjuntas a user/group/role).
- `Principal` se usa en **resource-based policies** (por ejemplo bucket policy de S3 o trust policy de un role).

---

## 5) Errores típicos de examen

1. Creer que `Principal` va en cualquier policy.
2. Olvidar que **un user puede tener permisos sin grupo**.
3. Suponer que grupos se pueden anidar.
4. No considerar que un `Deny` explícito gana sobre `Allow`.

---

## 6) Ejemplo práctico rápido

### Escenario

Tenés:
- Grupo `developers`: permite `s3:GetObject` en bucket de artifacts.
- Grupo `devops`: permite `cloudwatch:PutMetricData`.
- Usuario `ana` en ambos grupos.

### Resultado

`ana` puede leer artifacts en S3 y publicar métricas en CloudWatch.

Si agregás una política con `Deny` explícito a `s3:GetObject` sobre ese bucket, `ana` deja de poder leer aunque haya `Allow` en otro lado.

---

## 7) Checkpoint de memoria (sin mirar)

1. ¿Qué diferencia hay entre autenticación y autorización en IAM?
2. ¿Un user necesita obligatoriamente un grupo para tener permisos?
3. ¿Dónde se usa `Principal` y dónde no?
4. ¿Qué gana cuando hay conflicto: `Allow` o `Deny` explícito?

---

## 8) Identity-based vs Resource-based vs Trust policy (clave)

### Identity-based policy

- **Se adjunta a una identidad**: user, group o role.
- Define qué puede hacer ESA identidad sobre qué recursos.
- Acá el principal es implícito (la identidad adjunta), por eso no lleva `Principal`.

### Resource-based policy

- **Se adjunta al recurso**: por ejemplo bucket S3, cola SQS, key KMS.
- Define quién (`Principal`) puede hacer qué sobre ese recurso.
- Muy usada para acceso cross-account a recursos.

### Trust policy de role

- Es un tipo especial de **resource-based policy** que está adjunta al role.
- No define permisos de acciones del negocio; define **quién puede asumir el role**.
- Usa `Principal` + normalmente `Action: sts:AssumeRole` (u operación STS equivalente).

### Regla práctica

- "¿Qué puede hacer Juan?" → identity-based.
- "¿Quién puede entrar a ESTE bucket/role?" → resource-based/trust.

### Evaluación de permisos (simplificada)

- AWS evalúa políticas aplicables en conjunto.
- Si existe `Deny` explícito, gana.
- Si no hay deny y existe al menos un `Allow`, se permite.
- En cross-account, deben permitir tanto la identidad del account origen como la policy del recurso destino.

---

## 9) Directrices y buenas prácticas de IAM (curso + validación AWS)

> Dejame verificar: estas prácticas están alineadas con la guía oficial de IAM de AWS.

- **No usar root para el día a día**; reservarlo para tareas excepcionales de cuenta.
- **Una persona, una identidad** (evitar cuentas compartidas).
  - En entornos modernos, AWS recomienda **federación/Identity Center** para humanos.
- **Asignar usuarios a grupos** y administrar permisos principalmente a nivel grupo.
- Definir y aplicar **política de contraseñas fuerte**.
- **Habilitar MFA** (especialmente en cuentas privilegiadas).
- Preferir **roles con credenciales temporales** para servicios/workloads (en lugar de claves largas).
- Usar access keys solo cuando sea necesario para acceso programático (CLI/SDK) y rotarlas.
- Auditar con **Credential Report** y revisar uso con **Access Advisor / last accessed**.
- **Nunca compartir usuarios IAM ni access keys**.

---

## 10) Modelo de responsabilidad compartida para IAM

### AWS (Security **of** the Cloud)

- Protege la infraestructura global subyacente.
- Opera seguridad de hardware, software base, red y facilities.
- Realiza análisis/controles de conformidad de su infraestructura.

### Cliente (Security **in** the Cloud)

- Gestiona **usuarios, grupos, roles y políticas** IAM.
- Habilita y exige **MFA** donde corresponda.
- Rota y protege credenciales (passwords, access keys, secretos).
- Aplica mínimo privilegio y revisa patrones de acceso.
- Configura correctamente permisos de recursos y escenarios cross-account.

### Regla de oro

AWS asegura **la nube**; vos asegurás **cómo usás la nube**.

---

## 11) Resumen del módulo IAM (ordenado para repaso)

- **Usuarios**: identidad para acceso humano o técnico (sin compartir).
- **Grupos**: contienen solo usuarios; simplifican asignación de permisos.
- **Políticas**: JSON que define permisos (`Allow`/`Deny`) sobre acciones/recursos.
- **Roles**: identidades asumibles con credenciales temporales (servicios, apps, cross-account).
- **Seguridad**: MFA + política de contraseñas + mínimo privilegio.
- **CLI/SDK**: acceso programático; priorizar credenciales temporales mediante roles.
- **Claves de acceso**: usar solo cuando no haya alternativa, rotar y auditar.
- **Auditoría**:
  - Credential Report → estado de credenciales.
  - Access Advisor/last accessed → uso real de permisos/servicios.

---

## 12) Trampas de examen frecuentes (de este bloque)

1. Confundir trust policy con permisos de acciones del rol.
2. Suponer que “usuario físico = IAM user” siempre (hoy AWS prioriza federación para humanos).
3. Creer que Access Advisor reemplaza Credential Report (se complementan).
4. Olvidar que en cross-account se necesitan permisos de ambos lados.
