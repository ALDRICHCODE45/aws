# Lambda — VPC, acceso a Internet, NAT y Endpoints

Resumen corto para examen.

---

## Regla principal

Cuando conectas una función **AWS Lambda** a una **Virtual Private Cloud (VPC)**, la función pasa a tu red privada.
Por defecto, **no tiene salida a Internet**.

---

## Puntos clave

- Poner Lambda en una subred pública **no** le asigna IP pública.
- Por eso, “subred pública” no significa Internet automático para Lambda.
- Para salir a Internet desde Lambda en VPC, usa:
  - Lambda en subred privada
  - ruta hacia **Network Address Translation Gateway (NAT Gateway)**
  - y el NAT hacia **Internet Gateway (IGW)**

---

## Acceso privado a servicios AWS (sin Internet)

Si solo necesitas servicios AWS como:

- **Amazon DynamoDB**
- **Amazon Simple Storage Service (Amazon S3)**

puedes usar **VPC Endpoints** y evitar NAT/Internet.

---

## Regla mental

- Lambda + VPC + API externa = **NAT Gateway**
- Lambda + VPC + solo servicios AWS = **VPC Endpoint**
- Lambda + VPC + base de datos privada (por ejemplo **Amazon Relational Database Service, Amazon RDS**) = conexión privada interna
