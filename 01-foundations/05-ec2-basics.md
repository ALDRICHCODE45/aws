# EC2 básico (ordenado + corregido)

Este apunte consolida tu primera clase de EC2 con precisión de examen.

---

## 1) Qué es EC2

- **EC2 = Elastic Compute Cloud** (no “claude”).
- Es un servicio de **Infrastructure as a Service (IaaS)**.
- Te permite lanzar y administrar **servidores virtuales** (instancias) bajo demanda.

Qué podés hacer alrededor de EC2:
- Computar (CPU/RAM) con distintos tipos de instancia.
- Almacenar datos (EBS / Instance Store / EFS según caso).
- Distribuir tráfico con Elastic Load Balancing.
- Escalar horizontalmente con Auto Scaling Group.

---

## 2) Qué elegís al crear una instancia

En el launch de EC2, típicamente definís:

1. **AMI** (sistema base + software preinstalado).
   - Linux / Windows; en algunos contextos macOS.
2. **Instance type** (vCPU, RAM, red, y perfil de hardware).
3. **Storage**:
   - **EBS**: bloque persistente.
   - **Instance Store**: almacenamiento efímero ligado al host.
4. **Network**:
   - VPC/subnet, IP privada y opcional IP pública.
   - ENI (interfaz de red) y capacidad de red según tipo de instancia.
5. **Security Group**:
   - Firewall virtual a nivel instancia (reglas inbound/outbound).
6. **Key pair** (acceso seguro, sobre todo en Linux por SSH).
7. **User data** (script/config de bootstrap al inicio).

---

## 3) Storage en EC2 (no mezclar conceptos)

### EBS (Elastic Block Store)
- Volumen de bloque para una instancia.
- **Persistente** (sobrevive stop/start; depende del flag delete-on-termination al terminar instancia).
- Uso típico: disco de sistema y datos persistentes.

### Instance Store
- Disco local físico del host.
- **Efímero**: se pierde al detener/hibernar/terminar instancia.
- Uso típico: caché, buffers, datos temporales recreables.

### EFS (Elastic File System)
- Sistema de archivos administrado (NFS) para montar en múltiples instancias.
- No es “disco local de EC2”; es un servicio aparte de almacenamiento compartido.

---

## 4) User Data (bootstrap) — punto CLAVE

`User Data` = datos/script que pasás al lanzar la instancia para automatizar tareas iniciales.

Usos comunes:
- actualizar paquetes,
- instalar software,
- bajar archivos/config común,
- dejar el server listo sin intervención manual.

Reglas importantes:
- En Linux, por defecto se procesa con `cloud-init`.
- **Por defecto corre en el primer arranque** (se puede configurar para ejecutar también en reinicios).
- En scripts Linux, se ejecuta como **root**.
- Si usás AWS CLI/API dentro de user data, conviene usar **IAM role (instance profile)** en lugar de credenciales hardcodeadas.

---

## 5) Trampas de examen frecuentes (EC2 básico)

1. Creer que EBS, EFS e Instance Store son equivalentes.
2. Olvidar que Instance Store es efímero.
3. Confundir Security Group (instancia) con NACL (subred).
4. Pensar que User Data necesariamente se ejecuta en cada reboot (por defecto no).
5. Usar credenciales estáticas en bootstrap en vez de IAM role.

---

## 6) Regla mental rápida

- **EC2** = servidor virtual.
- **AMI** = plantilla de arranque.
- **Tipo de instancia** = músculo (CPU/RAM/red).
- **EBS** = disco persistente.
- **Instance Store** = disco temporal rápido.
- **Security Group** = firewall de la instancia.
- **User Data** = “instalador automático” de primer arranque.

---

## 7) Security Groups en EC2 (sin confusiones)

### Qué son

- Un **Security Group (SG)** es un firewall virtual asociado a interfaces de red/instancias.
- Tiene reglas **inbound** (entrada) y **outbound** (salida).

### Reglas clave

- Son **stateful**:
  - si permitís una conexión de ida, la respuesta vuelve aunque la regla inversa no esté explícita.
- Solo tienen reglas **Allow** (no hay Deny explícito en SG).
- Una instancia puede tener **múltiples SG**; AWS evalúa la unión de reglas permitidas.
- Se aplican en tiempo real al modificar reglas.

### Ejemplo típico

Servidor web público:
- Inbound: permitir `80/443` desde `0.0.0.0/0`.
- SSH (`22`): permitir solo desde tu IP (no abierto al mundo).

---

## 8) IAM Role + Instance Profile (punto clave)

### Idea simple

- **IAM Role**: define permisos (qué puede hacer la app en AWS).
- **Instance Profile**: “contenedor” que EC2 usa para asociar ese role a la instancia.

### Cómo funciona

1. Creás un role con permisos (ej: leer `s3://mi-bucket-config`).
2. Lo asociás a la instancia (vía instance profile).
3. La instancia obtiene **credenciales temporales** automáticamente (IMDS).
4. Tu app/CLI usa esas credenciales sin guardar access keys en disco/código.

### Reglas de examen

- En EC2 se adjunta **1 role por instancia** (vía 1 instance profile activo).
- Evitá hardcodear `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` en scripts.
- Para `user data` que llama AWS APIs, usar role de instancia es la práctica correcta.

---

## 9) Entendiendo el user data típico del curso (LAMP/Apache)

Script visto:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hola Mundo desde $(hostname -f)</h1>" > /var/www/html/index.html
```

### ¿Qué hace cada línea y por qué?

- `#!/bin/bash`
  - Indica que es un script shell ejecutable por bash.
- `yum update -y`
  - Actualiza paquetes del sistema (mejora base de seguridad/compatibilidad).
- `yum install -y httpd`
  - Instala **Apache HTTP Server** (`httpd` = daemon web de Apache).
- `systemctl start httpd`
  - Levanta el servicio web inmediatamente.
- `systemctl enable httpd`
  - Hace que Apache arranque automáticamente en próximos boots.
- `echo ... > /var/www/html/index.html`
  - Crea/reemplaza la página principal que Apache publica por defecto.

### ¿Por qué `/var/www/html/`?

- En Amazon Linux con Apache, esa ruta es el **DocumentRoot** por defecto.
- DocumentRoot = carpeta desde donde el servidor web sirve archivos HTTP.
- Entonces, al escribir `index.html` ahí, cuando navegás a `http://IP-publica`, Apache devuelve ese archivo.

### Relación con Security Group

Aunque tengas Apache andando, desde Internet solo lo vas a ver si además:
- SG permite inbound `80` (HTTP) desde el origen que corresponda.
- La instancia tiene conectividad pública (por ejemplo IP pública + ruta al Internet Gateway).

### Gotchas rápidos

1. Si omitís `#!/bin/bash`, puede no ejecutarse como script shell.
2. `yum update -y` puede hacer más lento el primer arranque.
3. Si ejecutaras de nuevo el script con `>>`, apenda contenido; con `>` reemplaza el archivo.

---

## 10) Troubleshooting rápido: si ves "It works!" en lugar de tu HTML

Eso suele indicar que Apache está activo, pero tu `index.html` personalizado no quedó donde esperabas o no se escribió.

Checklist de diagnóstico:

1. **Shebang presente**
   - Asegurate de iniciar user data con `#!/bin/bash`.

2. **Redirección correcta en una sola línea**
   - Usar:
   ```bash
   echo "<h1>Hola Mundo desde $(hostname -f)</h1>" > /var/www/html/index.html
   ```
   - Si el `>` queda partido en otra línea, puede fallar o comportarse distinto según formato.

3. **Ruta final del archivo**
   - Verificar que exista `/var/www/html/index.html`.

4. **Logs de user data (cloud-init)**
   - Revisar `/var/log/cloud-init-output.log` para confirmar si hubo error de sintaxis.

5. **Recordar comportamiento de user data**
   - Corre por defecto en primer arranque.
   - Si editás user data después, no necesariamente vuelve a ejecutarse salvo configuración adicional.

6. **Página por defecto de Apache**
   - "It works!" puede venir de configuración/página default de Apache cuando no se está sirviendo tu `index.html` esperado.

---

## 11) Tipos de instancias (nomenclatura para examen)

Ejemplo: `c6g.large`

- `c` = **familia/serie** → **Compute Optimized** (prioriza CPU).
- `6` = **generación**.
- `g` = **opción/variante** → procesador **AWS Graviton (ARM)**.
- `large` = **tamaño** dentro de esa familia.

### Familias base que más aparecen

- `M` → **General Purpose** (equilibrio CPU/RAM/red).
- `C` → **Compute Optimized** (CPU intensa).
- `R` → **Memory Optimized** (RAM intensa).
- `T` → **Burstable** (baseline + créditos de CPU, típico dev/test/cargas variables).

### Opciones/variantes comunes (aprender patrón, no lista infinita)

- `g` → Graviton (ARM)
- `i` → Intel
- `a` → AMD
- `d` → incluye **instance store** local
- `n` → networking/EBS optimizado

### Regla para que no te parezca ambiguo

- El tamaño (`large`, `xlarge`, `2xlarge`, etc.) es **relativo a cada familia/serie**.
- O sea: `xlarge` en `M` y `xlarge` en `C` no significan “la misma máquina”, porque la familia prioriza recursos distintos.
