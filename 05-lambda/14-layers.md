# Lambda — Layers (capas)

Resumen corto para examen.

---

## Qué es una Layer

ZIP separado del código de la función, que se adjunta a una o varias **AWS Lambda**.
Al arrancar, AWS descomprime las layers junto con el código en `/opt` dentro del contenedor.

---

## Dos usos principales

### 1. Runtimes personalizados

Lambda trae runtimes oficiales: Node.js, Python, Java, Go, Ruby, .NET.
Para correr otros lenguajes hay que traer el runtime como layer.

Ejemplos:

- **C++** → `aws-lambda-cpp`
- **Rust** → `aws-lambda-rust-runtime`

### 2. Externalizar dependencias

Sacar librerías pesadas del paquete de la función y ponerlas en layers compartidas.

---

## Sin layers vs con layers

**Sin layers:**

```
Paquete (30 MB)
├── lambda_function.py
├── heavy_library_1
└── heavy_library_2
```

Si otra Lambda usa las mismas libs, hay que duplicar.

**Con layers:**

```
Paquete Lambda 1 (20 KB)        Paquete Lambda 2 (60 KB)
└── lambda_function_1.py        └── lambda_function_2.py
        │                                │
        └──── Capa 1 (10 MB) ─── heavy_library_1
        └──── Capa 2 (30 MB) ─── heavy_library_2
```

Ambas Lambdas comparten las MISMAS layers. Sin duplicación.

---

## Beneficios

- Paquete de función chico → deploy más rápido.
- Cold start más liviano.
- Reutilización entre Lambdas.
- Cambios de código no resuben las librerías.

---

## Límites para examen

- Hasta **5 layers por función**.
- Tamaño total (función + layers descomprimidas) **≤ 250 MB**.
- Las layers se versionan: `arn:...:layer:nombre:VERSION`.
- Se pueden **compartir entre cuentas** (públicas o con permisos explícitos).
- Se montan en `/opt` dentro del contenedor.

---

## Regla mental

- Lenguaje no soportado por Lambda → **custom runtime en layer**.
- Librerías pesadas usadas por varias Lambdas → **layer compartida**.
- Cambios solo en código de la función → no resube layers.
