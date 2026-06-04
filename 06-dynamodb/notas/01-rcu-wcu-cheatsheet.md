# Cheatsheet RCU y WCU

## WCU

> 1 WCU = 1 escritura/s de hasta **1 KB**

```
WCU = escrituras/s × ceil(tamaño en KB)
```

## RCU

> 1 RCU = 1 lectura strongly/s de hasta **4 KB**

```
RCU = lecturas/s × ceil(tamaño / 4 KB) × factor
```

| Consistencia  | Factor |
| ------------- | ------ |
| Eventually    | ×0.5   |
| Strongly      | ×1     |
| Transactional | ×2     |

## Errores comunes

- Usar bloques de 1 KB para RCU → son de **4 KB**
- Olvidar el factor de consistencia
- No redondear hacia arriba → SIEMPRE ceil()
- Datos en minutos → convertir a /s PRIMERO
