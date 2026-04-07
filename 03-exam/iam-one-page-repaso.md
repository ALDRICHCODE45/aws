# IAM — One Page de Repaso (Examen)

## 1) Núcleo conceptual

- **IAM es global** (no por región).
- IAM resuelve 2 cosas:
  - **Autenticación**: quién sos.
  - **Autorización**: qué podés hacer.
- **Regla de oro de evaluación**: `Deny` explícito **SIEMPRE** gana.

---

## 2) Identidades y acceso

- **Root user**: solo tareas excepcionales de cuenta.
- **IAM user**: identidad puntual (evitar cuentas compartidas).
- **Group**: contiene solo usuarios (no anida grupos).
- **Role**: identidad asumible con credenciales temporales (servicios/apps/cross-account).

Buenas prácticas:
- MFA en cuentas privilegiadas.
- Mínimo privilegio.
- No compartir usuarios ni access keys.
- Preferir **roles + credenciales temporales** sobre claves de largo plazo.

---

## 3) Tipos de policy (pregunta clásica)

### Identity-based policy
- Se adjunta a user/group/role.
- Define qué puede hacer esa identidad.
- **No lleva `Principal`**.

### Resource-based policy
- Se adjunta al recurso (ej: S3 bucket policy).
- Define **quién (`Principal`)** puede hacer qué sobre ese recurso.

### Trust policy (de un role)
- Tipo especial de resource-based policy sobre el role.
- Define **quién puede asumir el role** (`sts:AssumeRole`).
- **No define permisos de negocio** (eso va en permission policies del role).

Regla mental:
- “¿Qué puede hacer X?” → identity-based.
- “¿Quién puede entrar a este recurso/role?” → resource-based/trust.

---

## 4) Cross-account (mínimo para que funcione)

Para A → B usando role en B:

1. **En B (destino)**: trust policy del role confía en principal de A.
2. **En A (origen)**: identity policy permite `sts:AssumeRole` al role ARN de B.

Sin una de esas dos, falla.

---

## 5) Herramientas IAM de auditoría

### Credential Report
- Reporte **a nivel cuenta** (CSV).
- Estado de credenciales: password, MFA, access keys.
- Útil para auditoría/compliance.

### Access Advisor (last accessed)
- Último acceso a servicios para users/roles/groups/policies.
- Útil para recortar permisos no usados.

Diferencia clave:
- Credential Report = **estado de credenciales**.
- Access Advisor = **uso real de permisos/servicios**.

---

## 6) Shared Responsibility en IAM

- **AWS (Security of the Cloud)**: protege infraestructura global.
- **Cliente (Security in the Cloud)**: gestiona IAM, MFA, credenciales y permisos.

Frase de examen:
> AWS asegura la nube; vos asegurás cómo la configurás y usás.

---

## 7) Trampas típicas de examen

1. Confundir trust policy con permisos del role.
2. Pensar que `Allow` puede ganar a `Deny` explícito.
3. Creer que Access Advisor reemplaza Credential Report.
4. Olvidar doble validación en cross-account (origen + destino).
5. Usar root como práctica normal (mala práctica).

---

## 8) Checklist pre-examen (30 segundos)

- [ ] ¿Identifiqué si es identity policy, resource policy o trust policy?
- [ ] ¿Hay algún `Deny` explícito?
- [ ] Si es cross-account, ¿están habilitados ambos lados?
- [ ] ¿Piden auditoría de credenciales (Credential Report) o uso de permisos (Access Advisor)?
