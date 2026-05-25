# Lambda — Reutilización del contexto de ejecución

Resumen corto para examen.

---

## Idea principal

**AWS Lambda** reutiliza el contenedor entre invocaciones cuando puede.
Todo el código **fuera del handler** se ejecuta UNA vez por contenedor y queda cacheado para las siguientes invocaciones.

---

## Cold start vs warm start

- **Cold start** (arranque en frío): primera invocación o contenedor nuevo.
  1. Descarga el código.
  2. Inicializa el runtime.
  3. Ejecuta el código fuera del handler (imports, init, variables globales).
  4. Llama al handler.
- **Warm start** (arranque en caliente): contenedor reutilizado.
  1. Lambda **descongela** el contenedor existente.
  2. Llama directo al handler.
  3. Variables globales y conexiones siguen vivas.

Entre invocaciones, el contenedor queda **congelado** (frozen), no destruido.

---

## Qué va FUERA del handler

Todo lo costoso de crear:

- Clientes del **AWS SDK** (DynamoDB, S3, etc.).
- Conexiones a bases de datos.
- Carga de configuración o secrets.
- Inicialización de loggers.

Si esto va DENTRO del handler, lo pagás en CADA invocación.

---

## Ejemplo Go

```go
// MAL — cliente creado en cada invocación
func handler(ctx context.Context, event Event) error {
    cfg, _ := config.LoadDefaultConfig(ctx)
    db := dynamodb.NewFromConfig(cfg) // se recrea siempre
    // ...
}

// BIEN — cliente creado UNA vez, reutilizado
var db *dynamodb.Client

func init() {
    cfg, _ := config.LoadDefaultConfig(context.TODO())
    db = dynamodb.NewFromConfig(cfg)
}

func handler(ctx context.Context, event Event) error {
    // db ya está listo
}
```

---

## Punto de examen

- Inicializar SDK clients y conexiones FUERA del handler para reducir latencia.
- Variables globales persisten entre invocaciones del MISMO contenedor.
- No persisten entre contenedores distintos (Lambda escala horizontal).
- Para compartir estado entre contenedores: **Amazon ElastiCache**, **Amazon DynamoDB**, **Amazon S3**.
