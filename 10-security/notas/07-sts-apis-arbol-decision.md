# STS — Las 4 APIs (árbol de decisión)

## Árbol (la keyword del enunciado manda)

| ¿De dónde viene la identidad?                                | API                             |
| ------------------------------------------------------------ | ------------------------------- |
| Usuario/rol IAM dentro de AWS, **incluido cross-account**    | **`AssumeRole`**                |
| **SAML / Active Directory** corporativo (federación empresa) | **`AssumeRoleWithSAML`**        |
| **Web/social/OIDC**: Google, Facebook, Amazon, Cognito       | **`AssumeRoleWithWebIdentity`** |
| Usuario IAM **misma cuenta** + creds temporales **con MFA**  | **`GetSessionToken`**           |

## Disparadores

- "otra cuenta AWS / cross-account / asumir rol IAM" (sin federación) → **AssumeRole**
- "SAML / Active Directory / ADFS / on-prem corporativo" → **AssumeRoleWithSAML**
- "Google / Facebook / móvil / OIDC / Cognito Identity Pool" → **AssumeRoleWithWebIdentity**
- "MFA + el mismo usuario se da creds temporales" → **GetSessionToken**

## Trampa propia

Cross-account entre cuentas AWS = **AssumeRole** pelado.
NO es WithSAML (eso necesita un IdP SAML, no solo "otra cuenta").
La palabra clave está siempre en el enunciado: buscá SAML / web / MFA / cuenta.

## Bonus

- `GetFederationToken`: federar un usuario IAM para dar creds temp (menos común).
- Cognito Identity Pool usa `AssumeRoleWithWebIdentity` por debajo.

## Pregunta de prueba

Un usuario IAM quiere proteger con MFA sus llamadas programáticas (ej: StopInstances):
envía el código MFA y recibe credenciales temporales para esas llamadas. ¿Qué API?

A) `AssumeRole`
B) `AssumeRoleWithSAML`
C) `AssumeRoleWithWebIdentity`
D) `GetSessionToken`

<details><summary>Respuesta</summary>

**D** (`GetSessionToken`): usuario IAM de la misma cuenta + creds temporales con MFA.
Cuándo sería cada una:
- **AssumeRole** → asumir un rol (cross-account o dentro de AWS).
- **AssumeRoleWithSAML** → identidad desde SAML/Active Directory corporativo.
- **AssumeRoleWithWebIdentity** → login web/social (Google, Facebook, Cognito).
</details>
