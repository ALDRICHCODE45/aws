# Cognito — Amazon Cognito (Identity & Authentication)

Apunte de consolidación con foco de examen (DVA-C02).
Complementado con el proyecto backend Go + Gin en curso.

---

## 1) Mapa mental en 20 segundos

Cognito tiene **2 componentes principales**, no los mezcles:

```
Amazon Cognito
├── User Pools          → Autenticación (¿quién sos?)
│   ├── Sign-up / Sign-in
│   ├── Tokens (Access, ID, Refresh)
│   └── MFA, grupos, attributes
│
└── Identity Pools      → Autorización (¿qué podés hacer en AWS?)
    ├── Da credenciales AWS temporales
    ├── Integra con User Pools + proveedores externos
    └── Roles IAM con políticas granulares
```

**Regla de oro**: User Pool ≠ Identity Pool. Son cosas distintas con propósitos distintos.

---

## 2) User Pools — En detalle

### Qué hace
- Servicio de **gestión de usuarios** (sign-up, sign-in, MFA, recuperación de contraseña).
- Emite **tokens JWT** estándar (OpenID Connect / OAuth 2.0).

### Flujos de autenticación más comunes

| Flujo | Uso | ¿Quién participa? |
|---|---|---|
| **USER_PASSWORD_AUTH** | Login directo email/password (nuestro proyecto) | Cliente → Cognito |
| **AUTHORIZATION_CODE** | Web apps con redirect (OAuth 2.0) | Usuario → Cognito → App |
| **AUTHORIZATION_CODE_PKCE** | SPAs y mobile apps (más seguro) | SPA/mobile → Cognito |

**Para el examen**: Si la pregunta dice "machine-to-machine" o "backend-to-Cognito" → probablemente USER_PASSWORD_AUTH o CLIENT_CREDENTIALS.

### Tokens

| Token | Contiene | Para qué |
|---|---|---|
| **ID Token** | Claims del usuario (email, sub, phone_number, etc.) | Identidad del usuario |
| **Access Token** | Scopes y permisos del usuario | Autorizar acceso a recursos |
| **Refresh Token** | — | Obtener nuevos tokens sin re-login |

- Todos son **JWT** (JSON Web Tokens): header.payload.signature.
- Se pueden **verificar offline** usando el JWKS endpoint de tu User Pool.

### Atributos de usuario
- **Estándar**: email, phone_number, name, preferred_username, etc.
- **Custom attributes**: podés agregar los tuyos (ej: `custom:role`).
- `sub` → ID único e inmutable del usuario (clave foránea lógica en DynamoDB).

### Grupos de usuarios
- Agrupan usuarios para aplicar **diferenciación de acceso**.
- Se pueden asignar políticas o roles diferenciados basados en grupo.
- Ejemplo: `Admins`, `Editors`, `Viewers`.

### MFA (Autenticación Multi-Factor)
- Soporta: **SMS, TOTP (apps como Google Authenticator), email**.
- Se puede configurar como obligatorio o opcional.

### App Clients
- Cada App Client tiene un `client_id` y opcionalmente un `client_secret`.
- **App client público** (nuestro caso): no tiene secret, usa `InitiateAuth` directo.
- **App client confidencial**: requiere `SECRET_HASH` (HMAC con el client secret).

**Truco examen**: Si dice "no se puede exponer un client secret" (SPA, mobile) → App Client público.

---

## 3) Identity Pools (Federated Identity) — En detalle

### Qué hace
- Da **credenciales AWS temporales** (Access Key ID + Secret Access Key + Session Token) a usuarios.
- Estas credenciales duran **1 hora** por defecto (configurable).

### Cómo funciona
1. Usuario se autentica en un **Provider** (User Pool, Facebook, Google, SAML, OIDC, etc.).
2. Identity Pool recibe el token y lo valida.
3. Asigna un **IAM Role** basado en reglas (authenticated vs unauthenticated).
4. Devuelve credenciales AWS temporales al usuario.

### Auth vs Unauth Identities
- **Authenticated**: usuario logueado (tiene token válido).
- **Unauthenticated**: invitado sin login (también puede tener credenciales limitadas).

**Truco examen**: Si la pregunta dice "guest access to S3" o "allow unauthenticated access" → Identity Pool con unauth role.

### Proveedores soportados
- Amazon Cognito User Pool
- Login with Amazon
- Facebook, Google
- SAML 2.0 providers
- OpenID Connect (OIDC)
- **Desarrolladores personalizados** (Custom Provider)

---

## 4) Seguridad y buenas prácticas

### Almacenamiento de tokens (lado cliente)
- **Nunca** almacenar tokens en localStorage (vulnerable a XSS).
- Preferir **HttpOnly cookies** o secure storage del dispositivo.

### Token refresh
- Usar el **Refresh Token** para obtener nuevos tokens sin pedir credenciales.
- Los Access Tokens son de vida corta (1 hora por defecto).

### Resource servers y scopes
- En User Pools, podés definir **scopes personalizados** y asociarlos a resource servers.
- Ejemplo: `myapp.read`, `myapp.write`.

---

## 5) Cognito + DynamoDB (nuestro proyecto)

Arquitectura que estamos armando:

```
Cliente → POST /auth/login
        → Cognito User Pool (USER_PASSWORD_AUTH)
        → Devuelve tokens (access + ID + refresh)
        → Cliente usa ID Token en headers
        → GET /auth/me
        → Backend valida JWT vía JWKS
        → Devuelve sub + email
```

### Cognito NO valida cada request — distinción clave

Hay una sutileza importante que el examen puede usar para confundirte:

```
❌ Mal entendido:  Request → tu app → llama a Cognito → Cognito valida → respuesta
✅ Correcto:       Request → tu app → valida firma JWT localmente → respuesta
```

**Lo que hace tu backend en cada request:**
1. Descarga el JWKS de Cognito **una sola vez** (lo cachea en memoria).
2. Usa la **clave pública** del JWKS para verificar la firma del JWT localmente.
3. Si la firma es válida → confía en los claims (`sub`, `email`, `exp`, etc.).
4. Cognito **no recibe ninguna llamada** en ese proceso.

Cognito solo participa **al momento del login** (emite y firma el token).
Después, cada servidor valida la firma por su cuenta usando criptografía asimétrica (RS256).

**Por qué esto escala:**
- Validar una firma JWT es una operación matemática local → microsegundos, sin red, sin base de datos.
- Cualquier instancia puede validar cualquier token sin coordinarse con ninguna otra.
- El estado vive en el token, no en el servidor → diseño stateless puro.

**Mapeo a DynamoDB**:
- `sub` del token = `userId` (partition key en tabla `users`)
- Atributos adicionales: `email`, `name`, `createdAt`, etc.
- No hay "relación" formal Cognito↔Dynamo → el `sub` es el puente lógico.

---

## 6) Conceptos de examen que se te pueden cruzar

| Concepto | Respuesta rápida |
|---|---|
| ¿Dónde se almacenan usuarios? | User Pool |
| ¿Dónde se obtienen credenciales AWS? | Identity Pool |
| ¿Qué es el JWKS endpoint? | URL pública que contiene las claves públicas para verificar JWTs |
| ¿Puedo usar Cognito sin User Pool? | Sí, con Identity Pool + proveedores externos (Facebook, Google, etc.) |
| ¿Qué pasa si un token expira? | Se usa el Refresh Token para obtener nuevos tokens |
| ¿Puedo customizar la UI de login de Cognito Hosted UI? | Sí, con CSS custom y dominios propios |
| ¿Cognito Sync sigue vigente? | **No, está deprecated** (usar AppSync o tu propia lógica) |
| ¿Qué trigger permite validar datos antes del sign-up? | **Pre Sign-up Trigger** (Lambda trigger) |
| ¿Cómo fuerzo MFA para un grupo? | Configuración obligatoria de MFA en el grupo de usuarios |
| ¿Cuándo usar Cognito vs IAM directamente? | Cognito para usuarios finales; IAM para servicios/roles |

---

## 7) Checklist rápido de examen

1. ¿La pregunta habla de **login/signup de usuarios**? → User Pool.
2. ¿La pregunta habla de **dar permisos AWS a usuarios**? → Identity Pool.
3. ¿Menciona "credenciales temporales de AWS"? → Identity Pool.
4. ¿Pide "verificar JWT en backend"? → JWKS endpoint + validación de claims (`iss`, `aud`, `token_use`).
5. ¿Dice "personalizar flujo de autenticación con Lambda"? → Cognito Triggers.
6. ¿La pregunta mezcla "guest access + authenticated access"? → Identity Pool con auth + unauth roles.

---

## 8) Relación con otros servicios del examen

| Servicio | Cómo se conecta con Cognito |
|---|---|
| **API Gateway** | Authorizer Cognito (valida JWT en cada request) |
| **IAM** | Roles asignados por Identity Pool |
| **DynamoDB** | Almacenar datos de usuario complementarios |
| **Lambda** | Cognito Triggers (pre-signup, post-confirmation, custom auth) |
| **S3** | Identity Pool puede dar acceso directo a buckets (presigned URLs, políticas bucket) |