# Lambda y CodeDeploy - Deployment Strategies

## ¿Qué hace CodeDeploy con Lambda?

**Automatiza** el cambio gradual de tráfico entre versiones de Lambda usando **Aliases** con weighted routing.

**IMPORTANTE**: NO aumenta el tráfico total, **redistribuye** el tráfico existente entre v1 y v2.

---

## Estrategias de Deployment

### 1. Linear (Lineal)
Aumenta el % hacia nueva versión **gradualmente en intervalos fijos**:

**Linear10PercentEvery3Minutes:**
```
0 min  → 10% v2, 90% v1
3 min  → 20% v2, 80% v1  
6 min  → 30% v2, 70% v1
...
30 min → 100% v2
```

**Linear10PercentEvery10Minutes:**
```
0 min  → 10% v2, 90% v1
10 min → 20% v2, 80% v1
...
100 min → 100% v2
```

### 2. Canary (Canario)
Prueba con **% fijo** y después **todo de una**:

**Canary10Percent5Minutes:**
```
0-5 min → 10% v2, 90% v1  (prueba)
5+ min  → 100% v2         (todo o nada)
```

**Canary10Percent30Minutes:**
```
0-30 min → 10% v2, 90% v1
30+ min  → 100% v2
```

### 3. AllAtOnce
**Deployment inmediato** → 100% v2 (sin gradualidad)

---

## ¿Cómo funciona internamente?

CodeDeploy modifica los **pesos** del Alias PROD:

```json
// Durante Canary10Percent5Minutes

// Minuto 0-5
"PROD": {
  "v1": 0.9,  // 90% del tráfico
  "v2": 0.1   // 10% del tráfico
}

// Minuto 5+
"PROD": {
  "v1": 0.0,  // 0% del tráfico  
  "v2": 1.0   // 100% del tráfico
}
```

### Ejemplo con 1000 requests/min:

**Canary10Percent5Minutes:**
- Minuto 0-5: 100 requests → v2, 900 requests → v1
- Minuto 5+: 1000 requests → v2, 0 requests → v1

---

## Hooks de validación

### Pre-traffic Hook
- Ejecuta **antes** de enviar tráfico a nueva versión
- Ej: health check, validaciones de infraestructura

### Post-traffic Hook  
- Ejecuta **después** del deployment
- Ej: pruebas de integración, métricas de performance

**Si cualquier hook FALLA** → **Rollback automático** a versión anterior

---

## Configuración en SAM

```yaml
Globals:
  Function:
    AutoPublishAlias: live
    DeploymentPreference:
      Type: Canary10Percent5Minutes
      Hooks:
        PreTraffic: !Ref AliasUpdatePreHook
        PostTraffic: !Ref AliasUpdatePostHook
```

---

## Casos de uso para examen

### ¿Cuándo usar cada estrategia?

**Linear:**
- Deployment **muy conservador**  
- Aumentar gradualmente para detectar problemas temprano
- Aplicaciones críticas con baja tolerancia al error

**Canary:**
- **Probar con poco tráfico** primero
- Si no hay problemas → deployment completo
- Balance entre velocidad y seguridad

**AllAtOnce:**
- Deployment **rápido** sin validación gradual
- Aplicaciones no críticas
- Cuando tenés alta confianza en la nueva versión

---

## Para el examen

**Pregunta típica:** "Deployment gradual con rollback automático si hay errores"

**Respuesta:** CodeDeploy con Canary/Linear + Pre/Post traffic hooks

**Key facts:**
- CodeDeploy usa **Lambda Aliases** con weighted routing
- **Hooks** permiten validaciones automáticas  
- **Rollback automático** si hooks fallan
- Integración nativa con **SAM framework**
- **Redistribuye** tráfico existente, no lo aumenta