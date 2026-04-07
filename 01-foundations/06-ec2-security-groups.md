# EC2 Security Groups (SG) — ordenado + corregido para examen

Este apunte consolida tus slides de Security Groups con precisión técnica y enfoque de certificación.

---

## 1) Qué es un Security Group

- Es un **firewall virtual** asociado a las interfaces de red de recursos en una **VPC** (por ejemplo, instancias EC2).
- Controla tráfico:
  - **Inbound** (entrada a la instancia)
  - **Outbound** (salida desde la instancia)

Regla clave:
- Un SG tiene **solo reglas Allow** (no existe Deny explícito en SG).

---

## 2) Comportamiento importante (muy de examen)

- **Stateful**: si una conexión está permitida en un sentido, el tráfico de respuesta vuelve aunque no haya regla espejo.
- Una instancia puede tener **múltiples SG** y AWS evalúa la **unión** de reglas permitidas.
- Cambios en reglas SG se aplican en caliente (sin reiniciar instancia).

---

## 3) Valores por defecto al crear SG

- **Inbound**: sin reglas (todo bloqueado hasta autorizar).
- **Outbound**: regla allow-all por defecto (se puede restringir).

---

## 4) Qué se puede filtrar en una regla

- Protocolo (TCP/UDP/ICMP)
- Puerto o rango de puertos
- Origen/Destino:
  - CIDR IPv4/IPv6
  - Prefix list
  - **Otro Security Group** (muy usado entre capas)

---

## 5) Referenciar otros Security Groups (arquitectura limpia)

En lugar de abrir por IP, podés permitir por SG:

- Ejemplo clásico 3 capas:
  - SG-ALB permite 80/443 desde internet.
  - SG-WEB permite 80/443 **solo desde SG-ALB**.
  - SG-DB permite 3306 **solo desde SG-WEB**.

Beneficio:
- Menos acoplamiento a IPs, más seguridad y mantenibilidad.

---

## 6) Puertos clásicos (memorizar)

- `22` → SSH (Linux)
- `21` → FTP
- `22` → SFTP (sobre SSH)
- `80` → HTTP
- `443` → HTTPS
- `3389` → RDP (Windows)

---

## 7) Scope y ubicación lógica

- Un SG vive dentro de una **VPC** (y por tanto en una región específica de esa VPC).
- Se asocia a ENI/instancias de esa VPC.

---

## 8) Troubleshooting rápido (timeout vs refused)

### Caso A: **Connection timed out**
Suele ser problema de red/perímetro:
- SG inbound/outbound
- NACL
- Route table / Internet Gateway
- IP pública/EIP

### Caso B: **Connection refused**
Suele indicar que llegaste al host pero:
- el servicio no está escuchando,
- o está caído,
- o está en otro puerto.

> Ojo examen: no asumir “siempre SG”; primero distinguir timeout vs refused.

---

## 9) Buenas prácticas prácticas

1. No abrir SSH/RDP a `0.0.0.0/0` salvo laboratorio puntual.
2. Separar SG por responsabilidad (ej: SSH-admin, web-public, db-private).
3. Preferir referencias SG↔SG entre capas internas.
4. Principio de mínimo privilegio en puertos y orígenes.
5. Documentar cada regla con descripción.

---

## 10) Checklist mental de 20 segundos (examen)

1. ¿Inbound o Outbound?
2. ¿Puerto/protocolo correctos?
3. ¿Origen/destino correcto (CIDR o SG)?
4. ¿El síntoma es timeout (red) o refused (servicio)?
5. ¿Hay reglas demasiado abiertas?
