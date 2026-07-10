---
name: tester
description: Use when a feature or bugfix needs a test plan, when verifying work before closing or shipping it, when asked "cómo lo probamos" or to prove something works, when designing QA coverage for backend or frontend changes, or when running E2E/UI tests (Playwright or similar). Also use before declaring any feature "done" or "tested".
---

# Tester — plan de pruebas y ejecución con evidencia

## Principio

Un plan de pruebas se deriva de las **invariantes** de la feature (qué debe ser imposible que pase), no de su lista de pantallas o endpoints. La feature está probada cuando existe **evidencia ejecutada** de cada caso — nunca cuando "el código se ve bien".

**Violar la letra de estas reglas es violar su espíritu.**

## Flujo obligatorio

1. **Mapear invariantes** de la feature → lee `references/test-plan.md` y construye el plan con su plantilla.
2. **Ejecutar el plan**: directo si es chico; si hay áreas independientes, despliega subagentes → lee `references/subagent-briefs.md` para escribir cada brief.
3. **Si la feature tiene UI**, el plan SIEMPRE incluye casos de `references/ui-testing.md` (no solo backend).
4. **Veredicto**: reporta con la plantilla de `references/verdict.md`. Sin esa plantilla no hay cierre.

## Qué reference cargar

| Situación | Archivo |
|---|---|
| Diseñar el plan de pruebas de una feature | `references/test-plan.md` |
| Escribir instrucciones para subagentes de prueba | `references/subagent-briefs.md` |
| La feature toca front/UI/UX, responsive, formularios | `references/ui-testing.md` |
| Reportar resultados / decidir si la feature está probada | `references/verdict.md` |

## Reglas duras (sin excepciones)

1. **Prioridad fija de casos:** (1) adversariales/seguridad → (2) invariantes de negocio → (3) concurrencia e idempotencia → (4) happy path → (5) bordes. Nunca empezar por el happy path.
2. **Prueba de fuego por caso:** *"si borro la lógica de enforcement, ¿este test sigue verde?"* Si sí, el test no vale y se rediseña.
3. **Dependencias reales:** todo caso que ejerce una invariante (constraint de DB, RLS, cola, unicidad) corre contra Postgres/Redis/DB reales, no mocks.
4. **Evidencia o no pasó:** cada caso reporta comando ejecutado + salida relevante. "Pasó ✅" sin salida no cuenta como ejecutado.
5. **No se declara probada** una feature con casos de prioridad 1–2 fallidos o sin ejecutar.

## Racionalizaciones prohibidas

| Excusa | Realidad |
|---|---|
| "Es un cambio simple, no necesita plan" | Los cambios simples rompen invariantes ajenas. El plan puede ser de 3 casos, pero existe. |
| "El typecheck/build pasó" | Compilar no es probar. Cero casos ejecutados = cero evidencia. |
| "Ya lo probé manualmente mientras desarrollaba" | Sin evidencia reproducible no hay prueba. Repetir con comando/registro. |
| "El happy path funciona, con eso basta" | Los bugs caros viven en los casos 1–3. El happy path es lo último. |
| "Mockeo la DB para ir más rápido" | El mock no ejecuta el constraint/RLS que ES la garantía. Prueba contra lo real. |
