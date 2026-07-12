# Diseño de tablas — intake, tipos y naming

**Nada de CREATE TABLE sin intake.** El schema se deriva de las consultas y del ciclo de vida del dato, no al revés.

## Intake mínimo (responde o pregunta ANTES de diseñar)

| # | Pregunta | Por qué cambia el diseño |
|---|---|---|
| 1 | ¿Qué consultas van a leer esta tabla? (las 3-5 principales, con sus filtros y orden) | Define índices, desnormalización y si el JSON es aceptable. |
| 2 | ¿Cardinalidades? (1:1, 1:N, N:M; ¿cuántos hijos por padre típicamente?) | Define FKs, tablas puente y si conviene embeber. |
| 3 | ¿Volumen esperado a 1-2 años? (¿cientos, millones, cientos de millones de filas?) | Define tipos de ID, particionado, y qué migraciones serán posibles después. |
| 4 | ¿Es multi-tenant? ¿Cuál es la tenant key? | Toda tabla de tenant lleva `tenant_id`/`organization_id` NOT NULL + índice; para RLS y aislamiento en consultas, ver el skill `seguridad` si está instalado. |
| 5 | ¿Ciclo de vida? (¿se actualiza, es append-only, se borra, cuánto se retiene?) | Define soft-delete, tablas de historial/eventos, políticas de purga. |
| 6 | ¿Ratio lectura/escritura? (¿dashboard que lee mucho, o ingesta que escribe mucho?) | Define cuántos índices te puedes permitir. |
| 7 | ¿Quién más escribe/lee la DB? (otra app, ETL, humanos con SQL) | Si la app no es el único cliente, la validación en app NO protege nada: constraints obligatorios. |

Si el usuario no sabe una respuesta, escribe el supuesto explícito en el plan ("asumo <1M filas/año") — un supuesto escrito se puede corregir; uno implícito, no.

## Tabla de tipos — reglas exactas (Postgres; nota MySQL donde difiere)

| Dato | Tipo correcto | PROHIBIDO | Nota |
|---|---|---|---|
| Dinero | `NUMERIC(12,2)` o `BIGINT` en centavos | `FLOAT`/`REAL`/`DOUBLE` | Float acumula error de redondeo: 0.1 + 0.2 ≠ 0.3. En centavos si haces mucha aritmética. |
| Fecha-hora de evento | `TIMESTAMPTZ` | `TIMESTAMP` sin tz, `VARCHAR` | Siempre con timezone; guarda en UTC, convierte al mostrar. MySQL: `TIMESTAMP` (convierte a UTC) o `DATETIME` + convención UTC documentada. |
| Fecha sin hora (cumpleaños, fecha de corte) | `DATE` | `TIMESTAMPTZ` | Un cumpleaños no tiene timezone. |
| Enum de estados | `TEXT` + `CHECK (status IN (...))`, o tabla catálogo si crece | `VARCHAR` libre, enteros mágicos (1=activo) | El tipo `ENUM` nativo de Postgres es difícil de alterar; CHECK es igual de fuerte y migrable. |
| Booleano | `BOOLEAN NOT NULL DEFAULT ...` | `VARCHAR('si'/'no')`, `INT` 0/1 | Siempre con default: un boolean NULL es un tercer estado accidental. |
| ID primario | Decisión consciente: `BIGINT GENERATED ALWAYS AS IDENTITY` o `UUID` (v7 si está disponible) | `SERIAL` de 32 bits en tablas grandes, UUID v4 como PK en tablas de alta escritura | Identity/bigint: compacto, ordenado, enumerable (fuga de conteo si es público). UUID: no enumerable, generable en cliente; v4 fragmenta índices, v7 no. Si el ID viaja en URLs públicas → UUID o id público aparte. |
| Teléfono, CP, RFC, folio externo | `TEXT`/`VARCHAR` | `INT`/`BIGINT` | "Números" que no se suman ni restan son texto: preservan ceros a la izquierda y formatos. |
| Cantidades/contadores | `INT`/`BIGINT` con `CHECK (>= 0)` si aplica | — | Piensa el rango: `INT` se desborda en 2.1B. |
| Texto libre | `TEXT` | `VARCHAR(255)` por costumbre | En Postgres `VARCHAR(n)` no ahorra nada; limita solo si hay regla de negocio real (y con CHECK). MySQL: `VARCHAR` con límite sí importa para índices. |

## JSON vs columnas — regla de decisión

Una columna `JSONB` es válida SOLO si el dato cumple las tres:
1. No lo filtras ni ordenas en consultas frecuentes (o solo por claves que puedes indexar con GIN, y lo mediste).
2. No tiene FK hacia otras tablas.
3. Su estructura varía por fila (payloads de webhooks, atributos de producto heterogéneos, respuestas crudas de un LLM).

Todo lo demás — status, montos, fechas, referencias — es columna tipada con constraint. "Lo meto al JSON para no migrar" = deuda que pagarás con una migración de backfill mucho peor.

## Naming — convención fija

- Tablas: plural, `snake_case` (`orders`, `order_items`). Elige singular o plural una vez por proyecto y NO mezcles; si el proyecto ya tiene convención, síguela.
- Columnas: `snake_case`; FKs como `<singular>_id` (`order_id`, `tenant_id`).
- Booleanos: prefijo `is_`/`has_` (`is_active`, `has_invoice`).
- Timestamps de evento: sufijo `_at` (`created_at`, `paid_at`, `deleted_at`); fechas sin hora: `_on` o `_date`.
- Índices/constraints: nómbralos explícito (`idx_orders_tenant_created`, `uq_users_email`, `fk_items_order`) — los nombres autogenerados hacen ilegibles los errores y las migraciones de borrado.

## Columnas obligatorias en toda tabla de negocio

```sql
id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  -- o UUID: decisión del intake
tenant_id     BIGINT NOT NULL REFERENCES tenants(id),           -- si es multi-tenant, SIEMPRE
created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at    TIMESTAMPTZ NOT NULL DEFAULT now()                -- actualizada por trigger u ORM
```

## Soft-delete — decisión explícita, no default

- Usa `deleted_at TIMESTAMPTZ NULL` (no `is_deleted`: el timestamp te dice cuándo, y NULL = vivo).
- Costo real: TODAS las consultas deben filtrar `WHERE deleted_at IS NULL` (scope global del ORM) y los UNIQUE deben ser parciales: `CREATE UNIQUE INDEX uq_users_email ON users(email) WHERE deleted_at IS NULL;` — si no, no puedes recrear un usuario borrado.
- Hard-delete es correcto cuando hay obligación legal de borrar (LFPDPPP/GDPR) o el dato no tiene valor histórico. Decide por tabla y escríbelo.

## Historial y auditoría

Si el intake dice "necesitamos saber quién cambió qué" o el dato es un estado que transiciona (pedidos, tickets): tabla de eventos append-only (`order_events(order_id, event_type, payload jsonb, actor_id, created_at)`) además del estado actual. Reconstruir historia desde `updated_at` es imposible; agregarla después requiere que el pasado ya se haya perdido.
