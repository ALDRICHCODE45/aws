# CloudFront Functions vs Lambda@Edge vs Route 53 geo

## Tabla disparadores

| Pregunta dice | Servicio |
| ------------- | -------- |
| "modificar headers" / "redirect simple" / "menor costo/latencia" | **CloudFront Functions** |
| "lógica compleja" / "consultar DB" / "transformación grande" | **Lambda@Edge** |
| "geo-routing a backends/regiones FÍSICAMENTE distintos" | **Route 53 geo-routing** |
| "bloquear países enteros" | **WAF geo-match** |
| "contenido distinto por país desde el edge" (mismo backend) | **CloudFront Functions + Viewer-Country** |

## CloudFront Functions vs Lambda@Edge

| | CF Functions | Lambda@Edge |
| -- | ----------- | ----------- |
| **Dónde corre** | **Edge Locations** (600+, más cercano al user) | **Regional Edge Caches** (~13) |
| Lenguaje | **JavaScript** | Node.js + Python |
| Tiempo | <1 ms | hasta 30s |
| Eventos | Viewer Request/Response | + Origin Request/Response |
| Network access | ❌ NO | ✅ SÍ |
| Costo | **6x más barato** | Caro |

> CF Functions corre en el edge más cercano = latencia mínima. Lambda@Edge va a Regional Edge = más lejos = más latencia. Por eso CF Functions gana en "menor latencia" / "más cerca del user".

## Headers geo gratis de CloudFront

- `CloudFront-Viewer-Country` (ISO code: UK, PH, AR, US...)
- `CloudFront-Viewer-City`
- `CloudFront-Viewer-Country-Region`
- `CloudFront-Viewer-Time-Zone`

## 4 eventos CloudFront

```
Usuario → [Viewer Request] → cache → [Origin Request] → Backend
Usuario ← [Viewer Response] ← cache ← [Origin Response] ← Backend
```

## Trampa principal

Si CloudFront ya está en el medio → para geo-routing **no necesitás Route 53 geo**. Route 53 geo se justifica cuando tenés **múltiples backends físicos** (distintas regiones).

Mismo backend + lógica por país → **CloudFront Function** (menor esfuerzo).
