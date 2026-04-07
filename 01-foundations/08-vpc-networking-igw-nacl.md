# VPC Networking — IGW + NACL + flujo de tráfico (para examen)

Este apunte cierra la brecha entre Security Groups (instancia) y el resto del networking de una VPC: cómo entra y sale realmente el tráfico, qué es un IGW, qué es un NACL, y por qué abrir un SG a `0.0.0.0/0` NO garantiza nada.

---

## 0) Mental model: la VPC como edificio

- **VPC** = edificio de oficinas.
- **Subnet** = piso del edificio.
- **EC2** = oficina dentro del piso.
- **IGW** = puerta principal del edificio a la calle (Internet).
- **Route Table** = cartelería del hall que dice “para salir a la calle, esa puerta”.
- **NACL** = guardia en la entrada del PISO (subnet): revisa todo lo que entra y sale.
- **Security Group** = cerradura con memoria en la puerta de cada OFICINA (instancia).

Si una de estas piezas falla, el paquete no llega. Punto.

---

## 1) Internet Gateway (IGW)

### Qué es
- Componente **horizontalmente escalado, redundante y altamente disponible** gestionado por AWS.
- Permite comunicación entre la **VPC** y la **Internet pública**.
- Soporta **IPv4 e IPv6**.

### Reglas clave (muy de examen)
1. **Una VPC = máximo 1 IGW adjunto.**
2. El IGW **no genera costo** por sí mismo (sí el tráfico que pasa).
3. **Adjuntarlo NO es suficiente** para tener Internet. Hace falta:
   - Una **ruta** en la Route Table: `0.0.0.0/0 → igw-xxxx` (y `::/0` para IPv6).
   - La instancia debe tener **IP pública o Elastic IP**.
   - Los filtros (SG + NACL) deben permitir el tráfico.
4. **Qué es una subnet pública**: una subnet cuya Route Table tiene ruta hacia un IGW. No por nombre, no por tag. Por **ruta**.
5. **Qué es una subnet privada**: una subnet sin ruta a IGW (usualmente sale a Internet vía NAT Gateway).

### Error clásico
> “Abrí el SG a `0.0.0.0/0` pero no puedo acceder por SSH.”

Causa típica: no hay IGW, no hay ruta a IGW, o la instancia no tiene IP pública. **El Security Group no reemplaza al IGW**.

---

## 2) Network ACL (NACL)

### Qué es
- Firewall **a nivel de subnet** (no a nivel de instancia como el SG).
- Capa **adicional** de seguridad, que actúa **antes** que el SG en entrada y **después** que el SG en salida.

### Características clave (memorizar)
- **Stateless**: NO recuerda conexiones. Inbound y outbound se configuran **por separado**.
- **Reglas numeradas**: se evalúan de **menor a mayor número**, y se detiene en la primera coincidencia.
- Soporta **ALLOW y DENY explícitos** (el SG no tiene DENY).
- **Default NACL** (la que trae la VPC): permite TODO inbound y outbound.
- **Custom NACL** (creada a mano): por defecto **bloquea TODO** hasta que agregues reglas.
- Cada subnet está asociada a **una** NACL (si no se asigna otra, usa la default).

### Ejemplo de evaluación por número

```
Rule 100  ALLOW  TCP  22    from  1.2.3.4/32
Rule 110  DENY   TCP  22    from  0.0.0.0/0
Rule *    DENY   ALL  ALL   from  0.0.0.0/0   (implícita)
```

- `1.2.3.4` entra → matchea regla 100 (ALLOW).
- Cualquier otra IP → matchea regla 110 (DENY).

> Regla de oro: numerá con saltos (100, 110, 120...) para poder insertar reglas intermedias en el futuro.

### Stateless: el detalle que rompe todo

Como NACL no recuerda conexiones, hay que permitir el **tráfico de respuesta** explícitamente.

Cuando un cliente inicia una conexión, el servidor responde a un **puerto efímero** del cliente (rango alto, variable por SO):

- Linux moderno: `32768–60999`
- Windows reciente: `49152–65535`
- Rango seguro para AWS: `1024–65535`

Por eso, si usás NACL personalizada, típicamente vas a ver:

```
Inbound  100  ALLOW  TCP  22          from  tuIP/32
Outbound 100  ALLOW  TCP  1024-65535  to    0.0.0.0/0
```

El outbound NO es para que la instancia inicie conexiones: es para que pueda **responder** al cliente SSH.

---

## 3) NACL vs Security Group (tabla rápida)

| Característica         | Security Group          | NACL                       |
|------------------------|-------------------------|----------------------------|
| Nivel                  | Instancia (ENI)         | Subnet                     |
| Estado                 | **Stateful**            | **Stateless**              |
| Reglas                 | Solo ALLOW              | ALLOW **y** DENY           |
| Evaluación             | Todas las reglas        | Por número, de menor a mayor |
| Default (VPC default)  | Deny inbound, allow outbound | Allow all in/out      |
| Default (custom)       | Deny inbound, allow outbound | Deny all in/out         |
| Scope                  | Varias por instancia    | Una por subnet             |
| Se puede referenciar otro SG/NACL | Sí, SG↔SG   | No, solo CIDR              |

---

## 4) Flujo completo de tráfico (GRABATE ESTO)

Un cliente en Internet quiere hacer SSH a tu EC2 en subnet pública:

```
[Internet]
    ↓
[IGW]                          ← debe existir y estar attached a la VPC
    ↓
[Route Table de la subnet]     ← debe tener 0.0.0.0/0 → igw-xxxx
    ↓
[NACL inbound de la subnet]    ← stateless, revisa entrada por número
    ↓
[Security Group inbound]       ← stateful, revisa entrada
    ↓
[EC2 con sshd escuchando]      ← el servicio tiene que estar arriba
    ↓
Respuesta:
[SG outbound]                  ← automático por stateful
    ↓
[NACL outbound]                ← OJO: debe permitir ephemeral ports (1024-65535)
    ↓
[Route Table → IGW → Internet]
```

Si cualquiera de estos 9 pasos falla → **no hay conectividad**.

---

## 5) Síntomas vs causa (diagnóstico rápido)

| Síntoma                 | Dónde mirar primero                                         |
|-------------------------|-------------------------------------------------------------|
| **Connection timed out** | Red/perímetro: IGW, Route Table, NACL, SG, IP pública      |
| **Connection refused**   | Servicio: sshd caído, puerto equivocado, firewall del SO   |
| **Permission denied**    | Autenticación: clave `.pem`, usuario, permisos del archivo |

Regla mental:
- **Timeout = el paquete no llega.**
- **Refused = el paquete llega pero nadie atiende.**
- **Denied = atiende pero te rechaza por credencial.**

---

## 6) Trampas frecuentes de examen

1. **SG stateful ≠ NACL stateless**. Si mezclás ambos, tenés que configurar ephemeral ports en NACL.
2. **Default NACL permite todo; custom NACL bloquea todo** hasta que agregues reglas.
3. **SG no tiene reglas DENY**. Si necesitás DENY explícito → NACL.
4. **IGW no reemplaza al Route Table**. Attach sin ruta no sirve.
5. **Una VPC solo tiene 1 IGW**.
6. **Subnet pública no es por nombre**, es por ruta a IGW.
7. **SG se puede referenciar entre sí**, NACL no (solo CIDR).
8. Para SSH abierto a Internet: no basta `0.0.0.0/0` en SG. Hace falta IGW + ruta + IP pública.

---

## 7) Buenas prácticas

1. Dejá la **default NACL** tranquila y usá SG para la mayoría de los controles.
2. Usá NACL solo cuando necesites **DENY explícito** a nivel subnet (ej: bloquear IPs maliciosas).
3. Si tocás una NACL custom, acordate de **ephemeral ports** en el sentido contrario.
4. Separá subnets públicas y privadas, y mantené recursos sensibles (DB, backend interno) en privadas.
5. Siempre que puedas, referenciá SG↔SG en lugar de CIDR para relaciones entre capas.

---

## 8) Checklist mental de 20 segundos (troubleshooting)

1. ¿La subnet tiene ruta a IGW en su Route Table?
2. ¿La instancia tiene IP pública o EIP?
3. ¿El SG inbound permite el puerto?
4. ¿La NACL inbound permite el puerto?
5. ¿La NACL outbound permite ephemeral ports para la respuesta?
6. ¿El servicio está escuchando en el puerto correcto?
7. ¿El síntoma es timeout, refused o denied?

Si pasás estos 7 puntos, el 95% de los problemas de conectividad los aislás en menos de 1 minuto.
