# Frameworks de decisión — principios con reglas operativas

Estos principios se aplican en orden cuando dos opciones compiten. Cada uno tiene su regla ejecutable y su excepción legítima. Si recomiendas contra un principio, declara cuál y por qué la condición del usuario lo amerita.

## 1. Aburrido primero

**Regla:** tecnología con ≥5 años en producción masiva > tecnología nueva, siempre que cubra el requisito. La innovación se gasta en el PRODUCTO, no en la infra.
**Test:** ¿puedes encontrar 3 postmortems/guías de operación de otros equipos con este componente? Si no, es demasiado nueva para un equipo chico.
**Excepción legítima:** la tecnología nueva elimina una categoría entera de trabajo (ej. un BaaS que ahorra construir auth+API+realtime para un MVP de validación).

## 2. Monolito modular primero

**Regla:** un solo deployable con módulos de límites claros (capas, dominios separados en el código). Microservicios solo con dolor DEMOSTRADO: equipos que se bloquean entre sí, un módulo con perfil de carga radicalmente distinto, o requisito real de escalado independiente — nunca por dolor anticipado.
**Matiz:** separar **API** y **worker de jobs** (mismo codebase, dos procesos) NO son microservicios; es higiene: protege la latencia del API de los trabajos pesados y es barato desde el día 1 si hay jobs (recordatorios, webhooks, envíos).
**Test para separar algo:** ¿puedes nombrar la métrica o el incidente que lo exige? Sin métrica, no se separa.

## 3. Managed > self-hosted (mientras el costo lo permita)

**Regla:** DB, colas, storage, email — gestionados por default. El tiempo de ops es el recurso más caro de un equipo chico: 4 horas/mes de mantenimiento valen más que la diferencia de precio.
**Punto de quiebre:** cuando el costo del servicio managed supera ~3-5× el self-hosted equivalente Y el equipo ya tiene capacidad de ops real (alguien cuyo trabajo incluye operar infra), se re-evalúa pieza por pieza.
**Nunca self-hostear** (equipo chico): base de datos de producción, email transaccional, nada con datos que no puedes perder.

## 4. Postgres como default

**Regla:** la base de datos relacional (Postgres el default por RLS, JSONB, extensiones y ecosistema) hasta que un requisito específico y medible la descarte. Cubre: relacional, documentos (JSONB), full-text search básico, colas simples, geodatos (PostGIS).
**Descartes válidos:** grafos profundos como operación central, series de tiempo a escala de millones de puntos/día, cache sub-milisegundo (Redis complementa, no reemplaza), search avanzado con relevancia (cuando el full-text nativo se quede corto — no antes).
**Anti-patrón:** agregar una segunda base de datos "por si acaso". Cada motor adicional es backups, monitoreo y expertise duplicados.

## 5. No agregar piezas hasta que una métrica lo pida — pero dejar los seams

**Regla:** colas, cache, CDN, read replicas: NO entran a la arquitectura inicial salvo que el intake muestre la necesidad (ej. integraciones con reintentos → cola desde el día 1). Cada pieza se agrega cuando una métrica cruza umbral, no cuando "se siente lento".
**Los seams sí se pagan hoy (son casi gratis):** acceso a datos detrás de repositorios (permite cache después), lógica de dominio fuera de los controllers (permite mover a worker después), IDs y contratos que no asumen un solo servidor, feature flags simples, fechas en UTC.
**Umbrales de referencia:** cache → cuando la misma query pesada domina el p95; cola → cuando un request hace >1 llamada externa o >2s de trabajo; CDN → cuando assets/tráfico geográfico lo muestre; réplicas → cuando lecturas saturen a la primaria (visible en métricas, no en miedo).

## 6. Optimizar para costo de cambio, no para el estado final

**Regla:** pregunta de diseño central: *"¿qué tan caro es cambiar esta decisión en 6 meses?"* — no *"¿aguanta 1M de usuarios?"*. La respuesta del intake #9 (lo más incierto del negocio) señala qué partes deben ser baratas de cambiar; invierte la flexibilidad AHÍ y en ningún otro lado.
**Anti-patrón:** abstraer todo "por si acaso" (el ORM-agnóstico, el multi-cloud preventivo). La flexibilidad genérica es costo sin beneficio; la flexibilidad dirigida por la incertidumbre declarada es inversión.

## 7. Regla de reversibilidad (cuánto análisis merece cada decisión)

**Irreversibles o muy caras de revertir** — merecen análisis profundo, comparación escrita y consulta al usuario:
- Base de datos principal y su modelo (single-tenant vs multi-tenant con RLS vs DB-por-tenant)
- Lenguaje/runtime del backend
- Modelo de identidad/auth (propio vs proveedor)
- Región/residencia de datos (si hay compliance)

**Reversibles** — se decide rápido con el default razonable y se sigue; cambiarlas después cuesta días, no meses:
- Librería de UI, proveedor de email, herramienta de logs/monitoreo, CDN, framework de CSS, proveedor de hosting (si el código no usa servicios propietarios)

**Regla:** el tiempo de análisis debe ser proporcional al costo de revertir. Debatir 2 días el proveedor de email es tan grave como elegir la DB en 5 minutos.

## Resolución de conflictos entre principios

Cuando dos principios chocan (ej. managed [3] vs presupuesto mínimo [intake #2]): **gana la fuerza dominante identificada en el flujo**. Ej.: fuerza dominante = costo mínimo → se acepta más riesgo operativo y se documenta ("sin réplica: una caída de DB implica restaurar backup, ~1h de pérdida máxima — aceptado por presupuesto").
