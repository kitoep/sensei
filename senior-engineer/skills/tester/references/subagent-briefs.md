# Briefs para subagentes de prueba

Un subagente sin brief estricto divaga: prueba lo que no es, suaviza fallos o "arregla" código que no debía tocar. El brief es un contrato — el subagente ejecuta y reporta, no interpreta.

## Estructura obligatoria del brief (las 6 secciones, en este orden)

```markdown
1. OBJETIVO (una línea): qué se está verificando y de qué feature.
2. ALCANCE:
   - SÍ: archivos/flujos/casos exactos que cubre.
   - NO: qué NO tocar ni probar (otras áreas las cubren otros agentes).
   - PROHIBIDO: modificar código de producción. Si un test requiere un fix, REPORTARLO, no aplicarlo.
3. PREPARACIÓN: comandos exactos de setup (levantar servicios, migrar, seed, credenciales/env que usar).
4. PASOS: numerados, cada uno ejecutable tal cual (comando o interacción concreta).
5. CRITERIO POR PASO: qué salida/estado = PASÓ y qué = FALLÓ. Incluir el efecto real a verificar (fila en DB, mensaje no enviado), no solo el status.
6. FORMATO DE SALIDA: la tabla de resultados exacta (abajo) + orden de reportar fallos TAL CUAL,
   con salida completa del error, sin suavizarlos ("casi pasa", "fallaría solo si...") y sin arreglarlos.
```

Formato de salida que el brief exige al subagente:

```markdown
| # | Caso | Resultado | Evidencia |
|---|------|-----------|-----------|
| 1 | <caso> | PASÓ / FALLÓ / NO EJECUTABLE | <comando + fragmento de salida relevante> |

Fallos (reproducción exacta):
- Caso #N: <comando> → <salida de error completa> → <qué estado quedó en DB/sistema>
No ejecutables: <caso y razón concreta (falta env, servicio caído, etc.)>
```

## Regla de paralelización

- Un subagente por **área independiente** (API, UI, concurrencia, permisos).
- Nunca dos subagentes tocando el mismo estado (misma DB de test mutable, mismo usuario seed, mismo puerto). Si comparten estado: secuencial, o datos/tenants separados por agente.
- Los casos de concurrencia van en UN solo agente (la carrera se orquesta dentro del test, no entre agentes).

## Ejemplo 1 — Brief de integración de API

```markdown
OBJETIVO: Verificar la invariante de no-doble-reserva del endpoint POST /bookings (feature agenda).
ALCANCE:
- SÍ: casos 2, 3 y 5 del plan (unicidad de slot, concurrencia, retry idempotente).
- NO: casos de UI ni de permisos (los cubre otro agente).
- PROHIBIDO: modificar código en src/. Si un test existente está mal, repórtalo.
PREPARACIÓN:
- docker compose up -d db redis
- npm run db:migrate:test && npm run seed:test
- Env: DATABASE_URL=<...> REDIS_URL=<...>
PASOS:
1. Ejecuta: npm test -- bookings.integration
2. Caso concurrencia: ejecuta el test que dispara 10 POST /bookings simultáneos (Promise.all)
   al slot seed "slot-A" con 10 clientes distintos.
3. Verifica en DB: SELECT count(*) FROM bookings WHERE slot_id='slot-A' AND status='CONFIRMED';
CRITERIO:
- Paso 2: exactamente 1 respuesta 201 y 9 respuestas 409. Cualquier otra distribución = FALLÓ.
- Paso 3: count = 1. count > 1 = FALLÓ CRÍTICO (doble reserva real), repórtalo primero.
SALIDA: tabla estándar + fallos con reproducción exacta.
```

## Ejemplo 2 — Brief de E2E de UI

```markdown
OBJETIVO: Verificar el flujo de creación de cita desde la UI (feature agenda) con Playwright.
ALCANCE:
- SÍ: flujo completo crear-cita, estados de error del formulario, doble click en submit.
- NO: lógica de disponibilidad del backend (cubierta por el agente de API). No probar otros módulos.
- PROHIBIDO: modificar componentes. Si un selector no existe, reporta "NO EJECUTABLE + selector faltante".
PREPARACIÓN:
- npm run dev (esperar a que responda http://localhost:3000/health)
- Usuario de prueba: <email>/<pass del seed>. Viewport default 1280x720.
PASOS:
1. Login → ir a /agenda → crear cita en slot visible → verificar toast de éxito Y que la cita
   aparece en el calendario (recargar página y confirmar que persiste).
2. Submit del formulario con fecha vacía → verificar mensaje de error VISIBLE junto al campo
   (no solo consola) y que NO se creó la cita (el calendario no la muestra tras recargar).
3. Doble click rápido en "Agendar" → verificar que se crea UNA sola cita (contar tarjetas tras recargar).
CRITERIO:
- Usa getByRole/getByTestId, nunca selectores CSS de clases. Esperas por estado (expect(locator)),
  nunca waitForTimeout. Si necesitas un sleep para que pase, el resultado es FALLÓ (flaky), no PASÓ.
- Cada paso: la aserción del efecto persistido (tras reload) decide PASÓ/FALLÓ.
SALIDA: tabla estándar + screenshot de cada fallo (guardar en test-results/ y citar ruta).
```

## Ejemplo 3 — Brief adversarial de permisos

```markdown
OBJETIVO: Verificar que el rol RECEPCIONISTA no puede eliminar servicios (permiso services.delete).
ALCANCE:
- SÍ: intentar la acción prohibida por TODAS las vías: API directa (Bearer), no solo la UI.
- NO: otros permisos ni otros roles.
- PROHIBIDO: modificar guards o seeds de permisos.
PREPARACIÓN: obtener token de recepcionista del seed; tener un servicio existente con id conocido.
PASOS:
1. DELETE /services/:id con token de recepcionista (Bearer directo, sin pasar por la UI).
2. Verificar en DB que el servicio SIGUE existiendo.
3. Verificar la RAZÓN del rechazo: body.message debe indicar permiso denegado —
   si el 403 viene de CSRF/throttler/ruta inexistente, el caso es FALLÓ (no se probó el permiso).
4. Repetir con token de ADMIN: debe ser 200 y la fila desaparecer (control positivo:
   demuestra que el paso 1 falló por el permiso y no porque el endpoint esté roto).
CRITERIO: PASÓ solo si (1) rechazado por la razón correcta, (2) fila intacta, (3) control positivo OK.
SALIDA: tabla estándar; si el paso 1 devuelve 200, es FALLO CRÍTICO — repórtalo primero con evidencia.
```

## Errores comunes al escribir briefs

- Brief sin sección NO/PROHIBIDO → el subagente "ayuda" arreglando código de producción.
- "Prueba que funcione el módulo X" sin pasos numerados → el agente inventa su propio plan.
- No exigir formato de salida → recibes prosa optimista imposible de auditar.
- No pedir el control positivo en casos adversariales → un 403 por una ruta rota se reporta como "permiso funciona".
- Repartir casos de la misma tabla/estado entre dos agentes paralelos → resultados contaminados.
