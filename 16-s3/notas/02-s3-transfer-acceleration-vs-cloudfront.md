# S3 Transfer Acceleration vs CloudFront (dirección del tráfico)

Trampa clásica DVA: confundir el servicio según la **dirección** del tráfico.

## S3 Transfer Acceleration — SUBIDA (upload) a larga distancia
- Acelera **uploads** (y downloads) a un bucket S3 desde ubicaciones **lejanas**.
- Usa las **edge locations de CloudFront** + la **red backbone de AWS** para mover datos rápido.
- **Simple**: solo cambiás al endpoint de aceleración
  (`bucketname.s3-accelerate.amazonaws.com`). Sin cambios de infra ni de cliente.
- Disparador: "subir datos a S3 desde oficinas/usuarios lejanos, latencia alta, solución simple".

## CloudFront — ENTREGA (download) hacia usuarios finales
- CDN: cachea contenido en edge locations para **servir/descargar** datos *hacia afuera*.
- Es de **salida** (delivery), NO para acelerar subidas a S3.

## Discriminador
| Necesidad | Servicio |
| --------- | -------- |
| Subir rápido a S3 desde lejos | **Transfer Acceleration** |
| Entregar/descargar contenido a usuarios finales | **CloudFront** |

## Trampas
- "Subir a CloudFront y luego al bucket" → MAL, CloudFront es para entrega/descarga, no upload.
- **Cross-Region Replication (CRR)** → replica **después** de subir; no arregla la latencia de upload.
- **VPN** → securiza la conexión, NO la acelera; agrega complejidad.

## Ganchos
Transfer Acceleration = ENTRADA (upload lejano). CloudFront = SALIDA (download/delivery).
Si el problema es "subir más rápido a S3 desde lejos" → Transfer Acceleration.

## Pregunta de prueba

Oficinas en América, Europa y Asia suben datos semanalmente a un bucket S3.
La latencia de red ralentiza las cargas. Necesitan la forma **más simple** de
mejorar los tiempos de **subida** sin cambios complejos. ¿Qué usás?

A) Bucket S3 local por región + Cross-Region Replication (CRR)
B) Conexión VPN gestionada de AWS
C) Subir a CloudFront y descargar desde la caché local al bucket S3
D) S3 Transfer Acceleration

<details><summary>Respuesta</summary>

**D**: S3 Transfer Acceleration (acelera uploads a larga distancia, solo cambiás el endpoint).
- **A** → CRR replica después de subir; no mejora la latencia de upload.
- **B** → VPN securiza pero no acelera.
- **C** → CloudFront es para entrega/descarga, no para acelerar subidas.
</details>
