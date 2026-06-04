# DynamoDB — PartiQL

Lenguaje de consulta compatible con SQL para DynamoDB. Te permite hacer SELECT, INSERT, UPDATE y DELETE con sintaxis SQL en vez de la API nativa.

---

## Qué permite

- Seleccionar, insertar, actualizar y eliminar datos con sintaxis SQL
- Consultar **varias tablas** de DynamoDB
- Alternativa más familiar si venís de SQL

## Ejemplo

```sql
SELECT OrderID, Total
FROM Orders
WHERE OrderID IN [1, 2, 3]
ORDER BY OrderID DESC
```

## Dónde se ejecuta

- Consola de administración de AWS
- NoSQL Workbench para DynamoDB
- API de DynamoDB
- AWS CLI

---

## Para el examen

- PartiQL es solo una **capa de sintaxis** — por debajo sigue usando Query/Scan con las mismas RCU/WCU
- NO hace magia: un SELECT sin PK sigue siendo un Scan caro
- Si la pregunta dice "forma familiar de consultar DynamoDB para desarrolladores SQL" → PartiQL
