# EC2 Storage Deep Dive (EBS, AMI, Instance Store, EFS)

Apunte de consolidación con foco de examen (DVA-C02 + fundamentos de arquitectura).

---

## 1) Mapa mental en 20 segundos

- **EBS** = disco de bloque **persistente** para EC2.
- **Instance Store** = disco local del host, **efímero** (alto rendimiento).
- **EFS** = sistema de archivos **compartido por red** (NFS) para múltiples clientes.
- **AMI** = plantilla de arranque de instancias (SO + software + block device mapping).

Si mezclás esto, te cae cualquier trampa de examen.

---

## 2) EBS — reglas clave

### Comportamiento
- Un volumen EBS y una instancia deben estar en la **misma AZ**.
- Una instancia puede tener **múltiples volúmenes** EBS.
- EBS persiste independiente del ciclo de vida de la instancia (salvo configuración de borrado al terminar).

### Root vs data volume al terminar instancia
- **Root volume**: suele tener `DeleteOnTermination=true` por defecto.
- **Data volumes** adjuntos: típicamente no se eliminan por defecto.
- Este comportamiento se puede cambiar.

### Snapshots
- Son copias de seguridad a nivel bloque.
- Permiten recrear volúmenes y mover datos entre AZ/Regiones (vía copia de snapshot/AMI).
- **Snapshot Archive** puede reducir costo hasta ~75% para acceso infrecuente (con reglas/limitaciones).
- Restaurar un snapshot archivado puede tardar **hasta 72 horas**.

---

## 3) Tipos de volúmenes EBS (no confundir)

### SSD
- **gp3/gp2**: uso general, balance costo/rendimiento.
- **io1/io2 (Provisioned IOPS)**: cargas críticas de baja latencia / IOPS sostenidas.

### HDD
- **st1**: throughput optimizado para acceso frecuente secuencial (logs, big data, DW).
- **sc1**: menor costo para acceso infrecuente (cold data).

### Regla de boot volume
- Boot en EBS: **gp2/gp3/io1/io2**.
- **st1/sc1 no** soportan boot volume.

### Trampa clásica
- `st1` != `sc1`
  - frecuente/throughput: **st1**
  - frío/infrecuente/barato: **sc1**

---

## 4) EBS Multi-Attach

### Qué es
Permite adjuntar un único volumen EBS a múltiples instancias EC2.

### Reglas importantes
- Hasta **16 instancias por volumen** (misma AZ).
- Aplica a **io1/io2** (no gp3/st1/sc1).
- Las instancias adjuntas tienen lectura/escritura.
- No se usa como boot volume.

### Riesgo operativo
- Para escrituras concurrentes, necesitás control de consistencia (filesystem/locking/fencing apropiado).
- No asumir que cualquier FS estándar multi-writer entre nodos te salva solo.

---

## 5) AMI — para qué sirve realmente

AMI = “molde” para lanzar instancias idénticas rápido.

Incluye:
- SO + paquetes + configuración
- metadatos de arranque
- block device mapping

### Flujo típico desde una EC2
1. Lanzar instancia base
2. Personalizar
3. Crear AMI (AWS crea snapshots EBS)
4. Lanzar nuevas instancias desde esa AMI

### Reglas de examen
- AMI es **regional**.
- Para usarla en otra región, hay que **copiar** la AMI.
- `No reboot` al crear AMI existe, pero puede comprometer integridad del filesystem.

---

## 6) Instance Store — cuándo conviene

### Qué es
Disco local físico del host de EC2, muy alto rendimiento I/O.

### Persistencia
- Persiste en **reboot**.
- No persiste en **stop / hibernate / terminate**.
- Si falla el hardware subyacente, podés perder datos.

### Casos típicos
- caché
- buffers
- scratch/temporal recreable

### Regla práctica
Si no podés perderlo, no lo dejes solo en Instance Store.

---

## 7) EFS — el concepto que desbloquea todo

EFS no es “otro disco de EC2”.

Es un **filesystem compartido por red (NFS) administrado por AWS** para que múltiples clientes
(EC2/ECS/EKS/Lambda/Fargate) lean/escriban archivos en común.

### Cuándo usar EFS
- múltiples instancias necesitan el **mismo** set de archivos
- home directories compartidos
- contenido común entre nodos

### Características
- Escala automática
- Alta disponibilidad (Regional, recomendado)
- Pago por uso

---

## 8) Pregunta trampa real: “DB necesita 310,000 IOPS”

Si te dan opciones: `gp2`, `io1`, `io2 Block Express`, `Instance Store` y piden **310k IOPS**:

- io2 Block Express (un volumen) llega hasta 256k IOPS (Nitro).
- Entonces, dentro de esas opciones, la mejor respuesta suele ser **Instance Store**.

Ojo: eso es lógica de examen por opciones disponibles; en arquitectura real entran más variables
(durabilidad, réplica, RPO/RTO, diseño de BD, etc.).

---

## 9) Checklist de decisión rápida

1. ¿Necesitás persistencia fuerte por instancia? → **EBS**
2. ¿Necesitás archivos compartidos entre varios nodos? → **EFS**
3. ¿Necesitás I/O local ultra-rápido y dato recreable? → **Instance Store**
4. ¿Necesitás clonar entornos idénticos rápido? → **AMI**
