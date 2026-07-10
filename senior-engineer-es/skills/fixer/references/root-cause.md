# Causa raíz (Fases 3–4)

## El punto donde explota NO es la causa

El stack trace te dice dónde EXPLOTÓ, no por qué. Un `NullPointerException` en la línea 40 casi nunca se arregla en la línea 40: se arregla donde el null se volvió posible. Arreglar en el punto de explosión (agregar un `if (x != null)`) es tratar el síntoma — el null sigue viajando y explotará en otro lado.

**Regla:** rastrea el valor/estado malo hacia ATRÁS hasta el punto donde se originó, y arregla ahí.

## 5 porqués aplicado a código — ejemplo completo

Síntoma: un cliente recibió dos recordatorios de la misma cita.

1. **¿Por qué llegaron dos mensajes?** Porque hay dos jobs de recordatorio para la misma cita en la cola.
2. **¿Por qué hay dos jobs?** Porque al reprogramar la cita se encola un job nuevo sin cancelar el anterior.
3. **¿Por qué no se cancela el anterior?** Porque el `jobId` incluye el timestamp de la cita, así que el job viejo y el nuevo tienen IDs distintos y el dedupe de la cola no los ve como duplicados.
4. **¿Por qué el jobId incluye el timestamp?** Porque se diseñó para "un job por horario", no "un job por cita".
5. **¿Por qué se diseñó así?** Porque no existía una regla escrita de idempotencia: cada feature inventa su esquema de jobId.

- Fix del síntoma (mal): borrar a mano los jobs duplicados.
- Fix de la causa (bien, nivel 3–4): `jobId` determinístico por cita (`reminder:<appointmentId>`), y la reprogramación reemplaza el job.
- Fix de la clase (mejor, nivel 5): convención documentada + helper único para generar jobIds, y test de idempotencia obligatorio para todo job nuevo.

Detente en el nivel donde el fix es accionable y proporcional; casi nunca es el nivel 1.

## Regla del patrón simétrico — los bugs viven en familias

Al confirmar la causa raíz, es OBLIGATORIO buscar el MISMO patrón defectuoso en el resto del repo antes de dar por cerrado el fix:

1. Describe el patrón en una frase greppeable. Ej.: "catch de error de unicidad dentro de una transacción", "token de tercero guardado sin cifrar", "fecha construida con TZ local".
2. Búscalo mecánicamente: `grep -rn "<patrón>" apps/ packages/ src/` — en TODOS los módulos/servicios/workers del monorepo, no solo donde explotó. El código compartido (`packages/`, `lib/`, `utils/`) cuenta doble: un bug ahí es un bug en todos sus consumidores.
3. Cada ocurrencia encontrada: o la corriges en el mismo fix, o documentas explícitamente por qué esa instancia no está afectada. No hay tercera opción ("luego lo veo" = el mismo bug reportado en 2 semanas en el otro módulo).
4. Reporta el resultado de la búsqueda como parte del fix: "patrón buscado en X, Y, Z; encontrado y corregido en Y; Z no aplica porque...".

## Clasificar: ¿bug puntual o clase de bug?

Pregunta obligatoria antes de escribir el fix: **¿la arquitectura PERMITE que este bug vuelva a escribirse mañana?**

| Señal | Clasificación | Qué exige el fix |
|---|---|---|
| Typo, condición invertida, caso borde olvidado en UN lugar | Puntual | Fix + test de regresión |
| El mismo error puede escribirse en cualquier módulo nuevo (olvidar tenant_id, olvidar cifrar, olvidar idempotencia) | Clase | Fix + hacer la clase IMPOSIBLE o detectable |
| Ya apareció 2+ veces en el historial del repo | Clase (confirmada) | Igual que arriba, sin excusas |

Hacer la clase imposible, en orden de preferencia:
1. **Constraint de base de datos** (unique, check, FK, RLS): la garantía vive donde no se puede saltar.
2. **El sistema de tipos**: tipos que no permiten representar el estado inválido (ej. un tipo `EncryptedToken` distinto de `string`).
3. **Lint rule / verificación en CI**: detecta el patrón prohibido en cada PR.
4. **Helper único obligatorio**: una sola función correcta que todos usan (y lint que prohíbe la vía manual).
5. **Documentación/checklist**: el más débil; úsalo solo si 1–4 son imposibles, y dilo.

Si eliges no cerrar la clase (por costo/alcance), decláralo explícitamente al usuario como deuda: "este fix corrige el caso; la clase sigue abierta porque X".

## Fase 4 — El fix

- El fix ataca la causa identificada, en el punto de origen, no en el punto de explosión.
- Un fix por causa. Si en el camino viste otro bug, repórtalo o arréglalo en un cambio separado.
- Cero refactors oportunistas mezclados con el fix: el diff del fix debe leerse como "esto y solo esto arregla el bug".
- Si el fix es un workaround consciente (no ataca la causa), etiquétalo como tal ante el usuario y registra qué falta.
