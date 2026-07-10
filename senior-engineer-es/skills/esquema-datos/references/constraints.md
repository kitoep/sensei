# Constraints — la DB como garantía final

**Principio: la aplicación valida para dar buenos mensajes de error; la base de datos garantiza que el dato corrupto sea IMPOSIBLE.** La app tiene bugs, corre en N procesos concurrentes, y nunca es el único cliente de la DB (consola, ETL, scripts, el siguiente microservicio). Una regla que solo vive en la app es una sugerencia.

## Qué va en DB vs qué va en app

| Regla | ¿Constraint en DB? | ¿Validación en app? |
|---|---|---|
| "Este campo siempre existe" | `NOT NULL` — sí, siempre | Sí, para el mensaje de error del formulario |
| "El email es único" | `UNIQUE` — sí, siempre (la app NO puede garantizarlo: race entre el SELECT-check y el INSERT) | Sí, para UX; y maneja la violación del unique como caso esperado |
| "La orden pertenece a un cliente que existe" | `FOREIGN KEY` — sí, siempre | Normalmente implícita vía ORM |
| "El total no es negativo", "status ∈ {…}" | `CHECK` — sí | Sí |
| "El descuento requiere aprobación de un gerente" | No (lógica de proceso) | Sí — reglas de flujo/permiso viven en app |
| "El formato del teléfono es bonito" | No (formato de presentación) | Sí — normaliza antes de guardar |

Regla rápida: si violarla deja la DB en un estado que ningún flujo legítimo produce → constraint. Si depende de quién/cuándo/cómo → app.

## Catálogo con reglas exactas

### NOT NULL — el default, no la excepción
Toda columna es `NOT NULL` salvo que NULL tenga un significado de negocio que puedas decir en voz alta ("`deleted_at` NULL = está vivo", "`paid_at` NULL = no pagada"). "Por si acaso no lo mandan" no es un significado: eso es un default o un error de la app.

### UNIQUE — donde la app matemáticamente no puede
Todo "no puede haber dos X con el mismo Y" es un `UNIQUE`/índice único. Casos que siempre se olvidan:
- Unicidad compuesta y por tenant: `UNIQUE (tenant_id, email)` — el email se repite entre tenants, no dentro.
- Con soft-delete, unique parcial: `CREATE UNIQUE INDEX uq_users_email ON users(tenant_id, email) WHERE deleted_at IS NULL;`
- Idempotencia de escrituras externas (webhooks, jobs): `UNIQUE (provider, external_id)` convierte el evento duplicado en un error controlado en vez de una fila doble.

### FOREIGN KEY — con decisión explícita de ON DELETE
Toda columna `*_id` que referencia otra tabla lleva FK. La parte que separa al senior: elegir `ON DELETE` a propósito, por relación:

| ON DELETE | Úsalo cuando | Ejemplo |
|---|---|---|
| `RESTRICT` (default sano) | Borrar el padre con hijos es un error de negocio | No se borra un cliente con órdenes |
| `CASCADE` | El hijo no tiene sentido sin el padre | `order_items` al borrar `orders` |
| `SET NULL` | La relación es opcional y el hijo sobrevive | `assigned_agent_id` al borrar el agente |

`CASCADE` en cadenas largas es una bomba: audita qué arrastraría antes de ponerlo. Y toda columna FK necesita índice (ver `indices.md`).

### CHECK — invariantes de fila
```sql
CHECK (amount >= 0),
CHECK (status IN ('pending','paid','shipped','cancelled','refunded')),
CHECK (ends_at IS NULL OR ends_at > starts_at),
CHECK ((paid_at IS NULL) = (payment_method IS NULL))   -- campos que van juntos o ninguno
```
Estados/enums SIEMPRE con CHECK (o tabla catálogo + FK si el conjunto lo edita el negocio). Agregar un estado nuevo = alterar el CHECK en una migración: eso es una feature, no una molestia — te obliga a revisar cada lugar que hace switch sobre el estado.

### DEFAULT — para columnas de máquina, no de negocio
`DEFAULT now()` en timestamps, `DEFAULT false` en flags, `DEFAULT 'pending'` en status inicial. NO uses default para esconder que la app olvidó mandar un dato de negocio (un `DEFAULT 0` en `amount` convierte un bug en una venta gratis silenciosa).

## Agregar constraints a tablas que YA tienen datos

1. Mide la violación actual: `SELECT count(*) FROM t WHERE <viola la regla>;`
2. Limpia o migra esas filas (script por lotes, decisión de negocio documentada).
3. Agrega el constraint `NOT VALID` y luego `VALIDATE CONSTRAINT` (Postgres) para no lockear (ver `migraciones.md`).

Si el paso 1 da > 0 y "no hay tiempo de limpiar": el constraint NO se pospone indefinidamente — se crea `NOT VALID` (protege filas nuevas desde ya) y la limpieza queda como tarea con dueño.

## Manejo en la app

Los constraints van a disparar errores (unique violation, FK violation): la app los captura y traduce a mensajes útiles. Un 500 crudo por unique violation no es razón para quitar el constraint — es razón para manejar el error. En flujos concurrentes usa el constraint como mecanismo: `INSERT ... ON CONFLICT DO NOTHING/UPDATE` en vez de SELECT-then-INSERT.
