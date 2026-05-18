# Lambda — CloudFront Functions vs Lambda@Edge

Resumen corto para examen.

---

## Personalización en el borde (Edge)

Con **Amazon CloudFront** puedes ejecutar lógica cerca del usuario para reducir latencia.

CloudFront ofrece dos opciones:

- **CloudFront Functions**
- **Lambda at Edge (Lambda@Edge)**

---

## Eventos donde se ejecutan

### CloudFront Functions

Solo en eventos del espectador (viewer):

- **Viewer Request**
- **Viewer Response**

### Lambda at Edge (Lambda@Edge)

En 4 eventos:

- **Viewer Request**
- **Origin Request**
- **Origin Response**
- **Viewer Response**

---

## Diferencia práctica

### CloudFront Functions

- Lógica muy ligera y ultra rápida
- JavaScript
- Muy alta escala y menor costo
- No acceso a red, sistema de archivos ni cuerpo de request

### Lambda at Edge (Lambda@Edge)

- Lógica más completa
- Node.js o Python
- Más tiempo y memoria disponibles
- Sí permite red, sistema de archivos y acceso al cuerpo de request

---

## Casos de uso

### CloudFront Functions

- Normalizar clave de caché
- Modificar cabeceras
- Reescritura o redirección de URL
- Autorización liviana (por ejemplo JSON Web Token, JWT)

### Lambda at Edge (Lambda@Edge)

- Lógica con más tiempo de ejecución
- Uso de bibliotecas de terceros
- Llamadas a servicios externos
- Procesamiento con acceso a body HTTP

---

## Puntos de examen

- CloudFront Functions: solo viewer request/response.
- Lambda@Edge: viewer + origin (4 eventos).
- Lambda@Edge se asocia desde **US East (N. Virginia) (us-east-1)**.
- Para lógica simple/sensible a latencia: CloudFront Functions.
- Para lógica compleja con red/body/librerías: Lambda@Edge.
