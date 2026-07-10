---
name: esquema-datos
description: Use when designing or modifying database schemas, creating tables or models, writing or reviewing migrations, adding columns or indexes, choosing column types or primary keys, or when the user says "crea la tabla", "agrega un campo", "necesito una migración", "cambia el tipo de la columna"; also before running any ALTER TABLE against a production database.
---

# Esquema-datos — schemas y migraciones que no rompen prod

## Principio

El schema es el contrato más caro de cambiar de todo el sistema. Un bug de código se corrige con un deploy; un schema mal diseñado se corrige con migraciones de datos, downtime y semanas de trabajo. Por eso: **la DB es la garantía final de integridad (no la app), toda migración asume que la app vieja sigue corriendo mientras migras, y ningún diseño empieza sin conocer las consultas que lo van a leer.**

**Violar la letra de estas reglas es violar su espíritu.** No hay tablas "demasiado simples" ni migraciones "demasiado obvias" para el proceso.

## Protocolo (carga el reference ANTES de actuar en su fase)

| Situación | Qué haces | PUERTA para avanzar | Reference |
|---|---|---|---|
| Diseñar tabla/modelo nuevo | Intake de consultas, volumen, tenancy y ciclo de vida; luego tipos y naming por regla | Respondiste el intake (o lo preguntaste) y cada tipo de columna se justifica con la tabla de tipos | `references/diseno.md` |
| Cualquier ALTER/migración | Clasificar la operación (segura vs peligrosa) y aplicar el procedimiento expand/contract | La migración corre con la app vieja viva, sin lock largo, y tiene plan de rollback escrito | `references/migraciones.md` |
| Definir columnas/relaciones | Declarar constraints en la DB: NOT NULL, FK, UNIQUE, CHECK | Cada regla de negocio inviolable tiene su constraint en DB, no solo validación en app | `references/constraints.md` |
| Consultas lentas o tabla nueva con lecturas | Derivar índices del patrón de consulta real | Cada índice se traza a una consulta concreta y lo verificaste con EXPLAIN | `references/indices.md` |

Un cambio típico toca varias filas: tabla nueva = diseño + constraints + índices; columna nueva en prod = diseño (tipo) + migración + constraint.

## Red flags — DETENTE si te descubres pensando esto

- "Creo la tabla y luego vemos qué consultas hará la app" → el schema se diseña DESDE las consultas. Intake primero.
- "Renombro la columna en la migración" → rename in-place rompe la app vieja desplegada. Expand/contract.
- "La app ya valida eso, no necesito el constraint" → la app tiene bugs, corre en paralelo y no es el único cliente de la DB. El constraint va.
- "`FLOAT` para el precio está bien" → dinero es `NUMERIC`/enteros en centavos. Siempre.
- "Le pongo `NOT NULL` directo a la tabla" → en una tabla grande eso es un lock/scan completo. Procedimiento por fases.
- "Agrego un índice a cada columna por si acaso" → cada índice cuesta en cada escritura. Índices se trazan a consultas.
- "Es una tabla chica, no importa" → las tablas chicas de hoy son las de 50M de filas que nadie puede migrar mañana.

## Racionalizaciones conocidas

| Excusa | Realidad |
|---|---|
| "No sé el volumen futuro, diseño genérico" | No saber es una respuesta del intake: pregunta o declara el supuesto por escrito. Genérico = decisiones implícitas que nadie revisó. |
| "Expand/contract es mucho trabajo para un rename" | Un rename directo con la app vieja corriendo = 500s en prod durante todo el deploy. El expand/contract toma 2 migraciones; el incidente toma tu noche. |
| "Los constraints hacen lenta la DB" | Un FK/CHECK cuesta microsegundos por escritura. Un dato corrupto cuesta un script de limpieza, un incidente con cliente y no poder confiar en tu propia data. |
| "Luego agrego los índices cuando esté lento" | "Lento" en prod = clientes afectados ya. Los índices de las consultas conocidas van desde el día 1; los especulativos, nunca. |
| "El ORM genera la migración, con eso basta" | El ORM genera el ALTER ingenuo (lock, rewrite, rename destructivo). Tú eres responsable de leer el SQL generado y clasificarlo con `references/migraciones.md`. |
| "Guardo todo en un JSON y ya" | JSON es para datos sin esquema estable que no filtras ni juntas. Todo campo que consultas, validas o relacionas es una columna. |
