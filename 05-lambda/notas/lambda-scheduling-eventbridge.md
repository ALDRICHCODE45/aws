# Lambda — Ejecución programada (cron) → EventBridge

## Regla de oro
"Lambda + horario / cron / cada X minutos / scheduled" → **EventBridge**
(Scheduled Rule o EventBridge Scheduler). SIEMPRE. Sin excepción.

## Por qué EventBridge es "lo más manejable y rentable"
- Manejable = **managed / serverless** → AWS lo opera, sin servidores.
- NO significa "la opción con menos pasos". Una regla cron = 3 clics / 4 líneas.
- Costo casi cero, cero infraestructura.

## Distractores y por qué caen
- ❌ "Programar directamente desde la consola de Lambda" → **NO EXISTE** ese botón.
  Se agenda con un trigger de EventBridge ("Add trigger → EventBridge").
  (Misma familia de trampa que VisibilityTimeout que descarta duplicados.)
- ❌ Task Scheduler de tu PC Windows → no es cloud, PC apagada = no corre.
- ❌ EC2 con cron job → funciona pero pagás server 24/7. Lo MENOS rentable.

## Errores propios en esta pregunta (DOS lecciones)
1. **Auto-sabotaje**: iba a marcar EventBridge (correcta) y la cambió por
   sensación de "es mucho trabajo". → REGLA: no cambiar la respuesta que el
   criterio eligió, salvo error CLARO de lectura.
2. **"Manejable" ≠ "menos trabajo"** → manejable = managed/serverless.

## Lema léxico
"manejable / sin administrar / rentable / sin servidores" = managed/serverless.
"scheduled / cron / cada N min" + Lambda = EventBridge.

## Pregunta de prueba

Necesitás ejecutar una función Lambda cada 30 minutos de la forma más manejable
y rentable, sin componentes adicionales. ¿Qué hacés?

A) Crear una regla de EventBridge que la dispare cada 30 min
B) Lanzar una EC2 con un cron job que la invoque
C) Usar el Programador de tareas de tu PC con Windows
D) Habilitar la programación directamente desde la consola de Lambda

<details><summary>Respuesta</summary>

**A** (EventBridge scheduled rule): managed, serverless, costo casi cero.
Cuándo sería cada una (por qué las otras caen):
- **EC2 con cron** → funciona pero pagás un server 24/7 = lo menos rentable.
- **PC con Windows** → no es cloud, PC apagada = no corre.
- **programar desde consola Lambda** → no existe ese botón (se agenda vía EventBridge).
</details>
