# IAM Security Tools — Credential Report y Access Advisor

Este bloque organiza y corrige los dos conceptos que trajiste del curso.

---

## 1) IAM Credential Report

### Qué es

Un reporte **a nivel cuenta** (CSV) que lista identidades IAM y estado de credenciales.

### Para qué sirve

- Auditoría y compliance.
- Detectar users sin MFA.
- Ver access keys activas/viejas/no usadas.
- Revisar rotación de credenciales.

### Datos típicos que muestra

- `password_enabled`, `password_last_used`
- `mfa_active`
- `access_key_1_active`, `access_key_1_last_used_date`
- equivalentes para `access_key_2`

### Gotcha de examen

- Se puede generar como máximo **1 vez cada 4 horas** (si ya existe uno reciente, devuelve el último).
- No te da visibilidad completa de TODOS los tipos de credenciales de servicios específicos.

---

## 2) IAM Access Advisor (last accessed)

### Qué es

Funcionalidad para ver **último acceso** a servicios permitidos y ayudar a recortar permisos.

### Corrección importante

No es solo “a nivel usuario”. También se usa para analizar:

- users
- roles
- groups
- policies

### Para qué sirve

- Identificar permisos concedidos pero no usados.
- Aplicar mejor principio de mínimo privilegio.
- Limpiar políticas sobredimensionadas.

### Gotcha de examen/operación

- Muestra intentos de acceso (no solo exitosos).
- Hay ventana de actualización (puede tardar en reflejar actividad reciente).

---

## 3) Diferencia rápida entre ambos

- **Credential Report**: foco en **estado de credenciales** (password, MFA, keys) de la cuenta.
- **Access Advisor / last accessed**: foco en **uso real de permisos/servicios** para ajustar políticas.

---

## 4) Mini escenario práctico

Tenés 40 IAM users.

1. Sacás **Credential Report** y detectás 8 users sin MFA + 5 keys antiguas.
2. Revisás **Access Advisor** en roles/users críticos y encontrás servicios no usados.
3. Quitás permisos no usados y forzás MFA en cuentas sensibles.

Resultado: menos superficie de ataque y políticas más limpias.

---

## 5) Checkpoint (sin mirar)

1. ¿Cuál herramienta usarías para auditar MFA y access keys?
2. ¿Cuál usarías para recortar permisos no usados?
3. ¿Por qué ambas se complementan para least privilege?
