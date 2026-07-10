# Migraciones seguras — expand/contract, locks y rollback

**Regla suprema: toda migración corre mientras la versión VIEJA de la app sigue viva** (deploy no atómico, rollback posible, réplicas rezagadas). Una migración que solo funciona con el código nuevo ya desplegado está mal diseñada.

## Doctrina expand/contract

Ningún cambio incompatible se hace en un paso. Siempre:

1. **EXPAND** — agrega lo nuevo sin tocar lo viejo (columna nueva, tabla nueva, índice nuevo). Compatible con app vieja y nueva.
2. **MIGRATE** — despliega código que escribe en ambos lados (dual-write) y backfill de datos históricos por lotes.
3. **SWITCH** — despliega código que lee de lo nuevo. Verifica (conteos, checksums) antes de seguir.
4. **CONTRACT** — solo cuando nada lee/escribe lo viejo (verifícalo: logs, grep en el repo, días de gracia), elimina lo viejo en una migración posterior separada.

Cada paso es un deploy independiente y reversible. Si el paso N falla, los pasos 1..N-1 siguen siendo correctos.

## Catálogo: operación peligrosa → procedimiento seguro

| Quieres hacer | Por qué es peligroso en un paso | Procedimiento seguro |
|---|---|---|
| Renombrar columna | La app vieja hace SELECT/INSERT con el nombre viejo → 500s todo el deploy | Expand: `ADD COLUMN nueva`; dual-write + backfill; switch de lecturas; contract: `DROP COLUMN vieja`. (Alternativa: no renombres — el nombre viejo con un comentario cuesta menos que 4 deploys.) |
| Cambiar tipo de columna | `ALTER TYPE` reescribe la tabla con `ACCESS EXCLUSIVE lock` (minutos en tablas grandes) | Columna nueva del tipo correcto + dual-write + backfill por lotes + switch + drop. Excepción segura en Postgres: ampliaciones sin rewrite (`VARCHAR(50)→TEXT`, `NUMERIC(10,2)→NUMERIC(12,2)`). |
| `NOT NULL` en tabla existente | Full scan con lock para validar | Postgres: 1) `ADD CONSTRAINT ck CHECK (col IS NOT NULL) NOT VALID;` 2) `VALIDATE CONSTRAINT ck;` (lock débil) 3) `ALTER COLUMN col SET NOT NULL;` (usa el CHECK, no re-escanea) 4) `DROP CONSTRAINT ck;`. Antes: backfill de los NULL existentes. |
| Columna nueva con `DEFAULT` | En Postgres ≥11 es seguro (default virtual). En Postgres <11 y MySQL viejos: rewrite completo | Verifica versión. Si hay rewrite: agregar columna NULL → backfill por lotes → set default → NOT NULL por fases. |
| Agregar FK a tablas con datos | Valida toda la tabla con lock | `ADD CONSTRAINT ... FOREIGN KEY ... NOT VALID;` y luego `VALIDATE CONSTRAINT ...;` en paso separado. |
| Crear índice | `CREATE INDEX` normal bloquea escrituras | `CREATE INDEX CONCURRENTLY` (fuera de transacción; si falla, queda INVALID: dropea y reintenta). MySQL/InnoDB: `ALGORITHM=INPLACE, LOCK=NONE`. |
| `DROP COLUMN` / `DROP TABLE` | Irreversible; algo que no viste aún lo lee (reportes, ETL, otro servicio) | Solo en fase contract, tras verificar cero lecturas. Paso previo barato: renombra a `_deprecated_<nombre>` o revoca permisos una semana antes — si nada truena, dropea. Backup/snapshot verificado antes. |
| Backfill masivo (`UPDATE tabla SET ...` sin WHERE) | Lock largo, transacción gigante, replica lag, tabla inflada | Por lotes: `UPDATE ... WHERE id BETWEEN a AND b` (1k-10k filas), commit por lote, pausa entre lotes, reanudable (guarda el último id procesado). NUNCA dentro de la transacción de la migración de schema. |
| Cambiar PK / estrategia de ID | Todo lo anterior junto | Trátalo como proyecto, no como migración: columna nueva + FKs nuevas + dual-write + switch coordinado. Cuestiona si de verdad es necesario. |

## Reglas de ejecución

- **Toda migración de schema es corta.** Schema (DDL rápido) y datos (backfill lento) van en migraciones/procesos SEPARADOS. Un backfill dentro de la migración = deploy colgado y lock eterno.
- **Postgres: setea timeouts al migrar** para que la migración muera antes que tirar prod:
  ```sql
  SET lock_timeout = '5s'; SET statement_timeout = '60s';
  ```
  Si el DDL no obtiene el lock en 5s (porque una query larga lo bloquea), falla y reintentas — en vez de encolar a TODAS las queries detrás de tu ALTER.
- **Lee el SQL que genera tu ORM/framework** (`--dry-run`, `sqlmigrate`, `--pretend`). El ORM no sabe cuántas filas tiene tu tabla ni qué versión de app está viva; tú sí.
- **Una migración = un propósito.** No mezcles crear tabla + backfill + drop de otra cosa.

## Plan de rollback — se escribe ANTES de correr

Para toda migración responde por escrito:
1. ¿Cómo se revierte? (migración down real y probada, o "roll forward" con una migración correctiva — los DROP no tienen down honesto: el down recrea la columna pero no los datos).
2. ¿La app vieja funciona DURANTE y DESPUÉS de esta migración? (si no: no cumple expand/contract, rediseña).
3. ¿Qué verifico después de correrla? (conteo de filas, `\d tabla`, la consulta clave con EXPLAIN, cero errores en logs por 10 min).

## Checklist puerta — antes de correr en prod

- [ ] SQL real leído (no solo el archivo del ORM).
- [ ] Clasificada cada operación con el catálogo de arriba; ninguna en la columna "peligroso en un paso".
- [ ] Tamaño de la tabla consultado (`SELECT reltuples::bigint FROM pg_class WHERE relname='...'`) — "chica" es un dato, no una intuición.
- [ ] `lock_timeout`/`statement_timeout` seteados (Postgres).
- [ ] Backfill separado, por lotes y reanudable (si aplica).
- [ ] Plan de rollback escrito (las 3 preguntas).
- [ ] Probada contra una copia/staging con volumen realista, no una DB vacía — en DB vacía TODO es instantáneo y no aprende nada.
