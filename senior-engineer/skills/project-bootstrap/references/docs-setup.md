# Gobernanza — dejar el proyecto operable por agentes sin que se degrade

Un proyecto mantenido con agentes de IA se degrada si las reglas viven en la memoria de una conversación. Se gobiernan con tres piezas commiteadas: **CLAUDE.md** (reglas no-negociables; o `AGENTS.md` si la herramienta usa esa convención), **docs/** (la spec: el QUÉ) y **el tracker** (plan y progreso: el CUÁNTO). Esta fase las crea.

## 1. CLAUDE.md — las reglas que ningún agente puede violar

Créalo en la raíz ANTES de la primera feature. No es documentación general: es la lista corta de reglas que, violadas, rompen el proyecto. Plantilla (adaptar con las decisiones del intake y foundations):

```markdown
# CLAUDE.md — Reglas del proyecto

[1 línea: qué es el proyecto y para quién.]
[Estructura del repo: apps/paquetes y qué hace cada uno.]

## Regla #1 — [LA invariante del proyecto] (NO NEGOCIABLE)
[La regla que jamás se viola, con el CÓMO exacto. Si es SaaS multi-tenant:
el aislamiento de datos entre tenants, de dónde se deriva el tenant_id,
la doble barrera (app + RLS), y el checklist para toda tabla/endpoint nuevo.]

## Reglas #2–#N
[Máximo 3-4 invariantes más del dominio, ej.: "no doble reserva",
"el bot no inventa", "jobs idempotentes". Solo lo que rompe el producto.]

## Convenciones de código
- Capas: [controller → service → repositorio, o las del stack elegido].
- Validación: nunca confiar en input del cliente; la UI no es enforcement.
- [Stack de UI permitido / prohibido introducir librerías nuevas.]

## Seguridad establecida (mantener)
[Lista de lo ya implementado que ningún cambio puede regresar:
auth default-closed, secretos cifrados, rate limiting, sin PII en logs...]

## Prioridad de tests
1. [La invariante #1], 2. [la #2], ... — y la regla: los tests de
invariantes son ADVERSARIALES (prueban que lo prohibido FALLA), no
confirmatorios. Prueba de fuego: "si borro el enforcement, ¿el test
sigue verde?" — si sí, el test no vale.

## Convenciones de trabajo
- Ninguna tarea se declara terminada sin sus pruebas ejecutadas y
  documentadas. Si una prueba falla, se reporta tal cual.
- Docs = spec (el QUÉ). Tracker = plan y progreso (el CUÁNTO). Si una
  tarea cambia el QUÉ, actualizar el doc es parte de su definition of done.
- Comandos: [dev, migrate, typecheck, test, build — los reales del repo].

## Scope MVP — NO implementar
[La lista de exclusión del intake, pregunta 3. Explícita.]
```

Reglas para escribirlo:
- **Corto y denso.** Cada línea debe cambiar el comportamiento de quien la lee. Si es información general, va en `docs/`, no aquí.
- Solo entran reglas que ya se decidieron (intake + foundations). No aspiraciones.
- Cuando un bug real revele una regla nueva ("nunca X dentro de una transacción"), se agrega a CLAUDE.md como parte del fix. Así el archivo acumula las cicatrices del proyecto y ningún agente repite el error.

## 2. docs/ — la spec (el QUÉ)

Estructura mínima:

```
docs/
├── README.md          ← orden de lectura y regla de precedencia
├── 00_intake.md       ← respuestas del intake (ya existe desde Fase 0)
├── 01_decisiones.md   ← cimientos aplicados/descartados con razón (Fase 1)
├── 02_producto.md     ← qué hace el producto: features, flujos, reglas de negocio
├── 03_esquema_bd.md   ← modelo de datos y sus invariantes
└── 04_stack.md        ← stack, hosting, deploy
```

Reglas:
- `docs/README.md` declara el **orden de lectura** y la **precedencia ante contradicción** (típicamente: producto > esquema BD > stack). Sin precedencia declarada, dos docs contradictorios paralizan a cualquier agente.
- Los docs describen el estado DESEADO (spec), nunca el progreso. Prohibido escribir "ya está hecho X" en un doc — eso es del tracker.
- Si una tarea cambia el QUÉ (producto, schema, permisos, stack), actualizar el doc correspondiente es parte de su definition of done, no un "después".

## 3. Tracker — el plan y el progreso (el CUÁNTO)

Puede ser un archivo (`TRACKER.md`), un board o un artifact — pero es UNO solo y manda sobre el orden de trabajo. Contenido mínimo por milestone: tareas con estado, criterio de terminado medible, y **evidencia de pruebas** (qué se corrió y qué salió).

Reglas:
- **Regla de oro: ninguna tarea se declara terminada sin sus pruebas ejecutadas y documentadas en el tracker.** "Terminé el código" no existe como estado; existe "pruebas en verde: [evidencia]".
- Ante contradicción sobre el plan u orden, gana el tracker (los docs ganan sobre el QUÉ).
- Al diseñar un milestone nuevo, cruzar su alcance contra las tareas ya listadas en el tracker ANTES de escribir el spec: toda tarea previa se ejecuta o se re-planifica explícitamente con el usuario — nunca se omite en silencio.

## Salida de la fase

`CLAUDE.md`, `docs/` con README + los docs mínimos, y el tracker con los milestones de la Fase 2 — todo commiteado. El proyecto queda listo para que cualquier agente, con cualquier modelo, trabaje sin degradarlo.
