# DynamoDB como Session State Store

"¿Dónde guardar estado de sesión serverless y compartido?" → **DynamoDB**

| Alternativa | Por qué NO |
|---|---|
| ElastiCache | También sirve, pero es en memoria (no serverless) |
| EFS | Se monta como disco de red en EC2, no es key-value |
| EBS / Instance Store | Cache LOCAL, no compartido entre instancias |
| S3 | Latencia alta, no pensado para objetos pequeños |
