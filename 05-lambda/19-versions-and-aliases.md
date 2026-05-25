# Lambda Versions y Aliases

## Versions (Versiones)

### $LATEST
- Versión de **desarrollo** (MUTABLE)
- Cuando trabajás en una función Lambda, editás $LATEST
- Es la única versión que se puede modificar

### Versiones numeradas
- Cuando estás listo para **publicar**, creás una versión
- Las versiones son **INMUTABLES** (no se pueden cambiar)
- Números crecientes automáticos: 1, 2, 3, etc.
- Cada versión tiene su propio ARN único

```
arn:aws:lambda:region:account:function:myFunc:$LATEST  (mutable)
arn:aws:lambda:region:account:function:myFunc:1        (inmutable)
arn:aws:lambda:region:account:function:myFunc:2        (inmutable)
```

### ¿Qué incluye una versión?
- **Código** + **Configuración completa**
- Memoria, timeout, variables de entorno, VPC, etc.
- Una vez creada, es una snapshot inmutable

---

## Aliases (Alias)

### ¿Qué son?
- **"Punteros"** a versiones específicas de Lambda
- Son **MUTABLES** (podés cambiar a qué versión apuntan)
- Tienen su propio ARN
- Nombres típicos: `DEV`, `TEST`, `PROD`

```
DEV Alias   → apunta a $LATEST
TEST Alias  → apunta a versión 3
PROD Alias  → apunta a versión 2
```

### Casos de uso:

**✅ Environments estables**
```bash
# Event source (S3, API Gateway) apunta al alias, no a versión directa
arn:aws:lambda:region:account:function:myFunc:PROD
```

**✅ Canary Deployments**
```
PROD Alias → 95% tráfico a versión 1
          → 5% tráfico a versión 2 (nueva)
```

**✅ Rollbacks rápidos**
```bash
# Si v2 falla, cambiar PROD alias de v2 → v1
aws lambda update-alias --function-name myFunc --name PROD --function-version 1
```

---

## Reglas importantes para examen

### ❌ Aliases NO pueden apuntar a otros aliases
```
PROD Alias → TEST Alias  # INCORRECTO
PROD Alias → Versión 1   # CORRECTO
```

### ✅ Event sources usan aliases
- S3 Event Notifications
- API Gateway
- CloudWatch Events
- **Siempre apuntan a ALIAS**, no versión directa

### ✅ Weighted routing (Canary)
```json
{
  "PROD": {
    "Version": "1",
    "Weight": 0.9
  },
  "PROD": {
    "Version": "2", 
    "Weight": 0.1
  }
}
```

---

## Casos de examen típicos

**Pregunta:** "Necesitás deployment gradual enviando 10% tráfico a nueva versión"
**Respuesta:** Alias con weighted routing (Canary deployment)

**Pregunta:** "Event source S3 debe apuntar a versión estable"
**Respuesta:** Configurar S3 trigger contra ALIAS, no versión directa

**Pregunta:** "Rollback rápido si nueva versión falla"
**Respuesta:** Cambiar alias de v2 → v1 (inmediato, sin reconfigurar triggers)

---

## Workflow típico

1. **Desarrollo** → Editar $LATEST
2. **Testing** → `CreateVersion` (v1), alias TEST → v1  
3. **Staging** → Alias STAGING → v1
4. **Production** → Alias PROD → v1
5. **New release** → `CreateVersion` (v2), PROD → 90% v1 + 10% v2
6. **Full deployment** → PROD → 100% v2
7. **Rollback** (si falla) → PROD → v1