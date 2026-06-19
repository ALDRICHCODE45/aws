# SAM local — credenciales para invocar servicios AWS

## Flujo estándar (2 pasos)

```bash
aws configure --profile sandbox        # crea perfil en ~/.aws/credentials
sam local invoke Func --profile sandbox  # SAM usa ese perfil
```

## Disparador

"Access denied" en `sam local invoke` cuando la Lambda llama a S3/DDB/etc → **falta `--profile`** o **falta configurar el perfil**.

## Trampas

| Opción que aparece | Por qué descartar |
| ------------------ | ----------------- |
| Credenciales en `Globals` del template.yml | Secretos en código versionado. Anti-pattern. |
| `--parameter-overrides` | Es para parámetros de CFN, NO credenciales. |
| Crear archivo local custom con env vars | No es el mecanismo estándar. SAM usa `~/.aws/credentials`. |
