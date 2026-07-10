# Índices — derivados de consultas, verificados con EXPLAIN

**Regla: cada índice se traza a una consulta concreta que puedes escribir; cada consulta frecuente se traza a un índice que la sirve.** Un índice sin consulta es costo puro (cada INSERT/UPDATE lo mantiene); una consulta frecuente sin índice es un full scan que crece con la tabla.

## Método (en este orden)

1. **Lista las consultas reales** de la tabla (del intake de `diseno.md` o del código: busca los `WHERE`, `JOIN`, `ORDER BY` sobre la tabla).
2. **Diseña el índice para la consulta**, con las reglas de abajo.
3. **Verifica con EXPLAIN** que la consulta lo usa. Sin EXPLAIN, el índice es una esperanza.

## Reglas de diseño

### Índice compuesto: el orden de columnas ES el diseño
Orden: **igualdad primero → rango/orden después**.

```sql
-- Consulta: WHERE tenant_id = ? AND status = ? ORDER BY created_at DESC LIMIT 20
CREATE INDEX idx_orders_tenant_status_created
  ON orders (tenant_id, status, created_at DESC);
```
- Sirve prefijos: también cubre `WHERE tenant_id = ?` solo. NO sirve `WHERE status = ?` sin tenant_id (no es prefijo).
- Un índice `(a, b)` hace redundante a un índice `(a)` — no tengas ambos.
- La columna de rango (`created_at > ?`, `ORDER BY created_at`) va AL FINAL; todo lo que esté después de un rango en el índice ya no filtra eficientemente.
- Multi-tenant: `tenant_id` es la primera columna de casi todos los índices, porque casi toda consulta filtra por tenant.

### Toda FK lleva índice
Postgres NO indexa FKs automáticamente (MySQL/InnoDB sí). Sin índice en la FK: los JOIN hacen scan y cada DELETE del padre escanea la tabla hija completa (con lock). Al crear una FK, crea su índice en la misma migración — o justifícalo por escrito.

### Índices parciales — para el subconjunto que consultas
Cuando siempre consultas el mismo pedazo de la tabla:
```sql
-- Solo consultas los pendientes; el 95% histórico no estorba en el índice
CREATE INDEX idx_jobs_pending ON jobs (created_at) WHERE status = 'pending';
-- Unique con soft-delete (ver constraints.md)
CREATE UNIQUE INDEX uq_users_email ON users (tenant_id, email) WHERE deleted_at IS NULL;
```
Más chico, más rápido, más barato de mantener. MySQL no los tiene: usa índice normal y acepta el costo.

### Otros casos con regla fija
- Búsqueda por texto libre (`ILIKE '%x%'`): un B-tree NO sirve. Postgres: `pg_trgm` + índice GIN, o full-text search. No agregues un índice normal "a ver si ayuda".
- Filtros dentro de JSONB: índice GIN sobre la columna o índice de expresión sobre la clave (`CREATE INDEX ... ON t ((payload->>'type'))`) — solo si la consulta existe y lo mediste.
- Baja cardinalidad sola (boolean, status con 3 valores) NO se indexa sola: casi nunca filtra suficiente. Va como columna de un compuesto o condición de un parcial.

## Costo de escritura — el presupuesto

Cada índice se actualiza en cada INSERT y en cada UPDATE que toque sus columnas. Guías:
- Tabla transaccional típica: 3-6 índices es normal; 10+ exige justificación consulta por consulta.
- Tabla de ingesta masiva (eventos, logs, mensajes): mínimos índices; los análisis complejos van a una réplica/warehouse.
- Antes de agregar un índice a una tabla caliente, lista los que ya tiene (`\d tabla` / `SHOW INDEX FROM tabla`) y busca: ¿uno existente ya sirve como prefijo? ¿dos existentes son redundantes entre sí?

## Verificación con EXPLAIN (la puerta)

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ... ;  -- Postgres; MySQL: EXPLAIN ANALYZE
```
Lee el plan y confirma:
- Aparece `Index Scan`/`Index Only Scan` usando TU índice — no `Seq Scan` sobre la tabla grande.
- El `ORDER BY ... LIMIT` lo resuelve el índice (no hay nodo `Sort` sobre miles de filas).
- `rows` estimadas vs reales no difieren por órdenes de magnitud (si sí: corre `ANALYZE tabla;`).

Trampas al verificar:
- En una DB con 100 filas el planner elige Seq Scan aunque tu índice sea perfecto (es lo correcto con 100 filas). Verifica contra volumen realista o al menos razona el plan esperado y déjalo escrito.
- `EXPLAIN` sin `ANALYZE` es la estimación; con `ANALYZE` ejecuta de verdad — cuidado con correrlo sobre UPDATE/DELETE en prod (envuélvelo en `BEGIN; ... ROLLBACK;`).

## En producción: crear sin bloquear

`CREATE INDEX CONCURRENTLY` siempre en tablas vivas (fuera de transacción; si falla queda `INVALID`: `DROP INDEX` y reintenta). MySQL/InnoDB: `ALGORITHM=INPLACE, LOCK=NONE`. Detalles y checklist en `migraciones.md`.

## Mantenimiento — índices que ya nadie usa

Al revisar performance, además de agregar, borra:
```sql
SELECT indexrelname, idx_scan FROM pg_stat_user_indexes
WHERE schemaname='public' ORDER BY idx_scan ASC;  -- idx_scan=0 tras semanas = candidato a DROP
```
No borres los que respaldan UNIQUE/PK aunque tengan 0 scans: esos son constraints, no aceleradores.
