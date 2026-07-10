# Taxonomía de evidencia — qué cuenta como "verificado" por tipo de tarea

## La escalera de evidencia

De menor a mayor. Reporta siempre el nivel MÁXIMO que alcanzaste, con nombre y apellido — nunca dejes que un nivel bajo se disfrace de uno alto.

| Nivel | Qué probaste | Qué NO probaste |
|---|---|---|
| 0. "Se ve bien" (leer el diff) | Nada | Todo |
| 1. Typecheck / lint / build | Que el código compila | Que hace algo correcto |
| 2. Tests unitarios pasan | Lo que los tests cubren | El flujo real, integración, estado |
| 3. Ejecución real del flujo | Que el camino ejercitado responde | Los caminos no ejercitados, el estado persistido |
| 4. Estado real consultado | El efecto observable en el sistema (DB, archivo, entorno vivo) | Nada — este es el estándar de "terminado" |

**Regla:** una afirmación de comportamiento ("funciona", "quedó", "responde") exige nivel 3 mínimo; una afirmación de efecto ("guardó", "migró", "desplegó", "envió") exige nivel 4. Nivel 1-2 solo autoriza a decir "compila / los tests pasan".

## Evidencia mínima por tipo de tarea

### Feature nuevo
- Ejecuta el flujo completo de punta a punta como lo haría el usuario (request real, comando real o UI real) — no solo la función en aislamiento. Pega la salida.
- Mínimo 3 casos: el feliz + 2 adversarios (input inválido, caso vacío/límite). "Probé el feliz" no autoriza "funciona"; autoriza "el caso feliz funciona".
- Si el feature persiste o envía algo: consulta el estado real (fila en DB, mensaje creado, archivo generado) y pega el resultado.

### Refactor
- Suite completa del área ANTES y DESPUÉS, con resúmenes pegados (`X passed, 0 failed` en ambos). Mismo verde antes y después = comportamiento preservado; verde solo después no prueba nada.
- Búsqueda de todos los usos del símbolo/contrato cambiado (`grep` pegado) — cada call-site actualizado o justificado.
- Ejecuta una vez el flujo principal que atraviesa el código refactorizado (nivel 3), no solo los tests.

### Cambio de config
- No leas el archivo que editaste: consulta la config EFECTIVA en el sistema corriendo (endpoint de config, variable impresa por el proceso, comando del framework tipo `config:show`). El archivo correcto con el proceso sin reiniciar es el falso verde clásico.
- Ejercita el comportamiento que la config controla y pega la diferencia observable.

### Deploy
- Consulta el entorno VIVO: versión/commit desplegado (`/version`, header, log de arranque), no el pipeline en verde. Pipeline verde = el deploy corrió, no que lo tuyo está arriba.
- Ejecuta contra el entorno desplegado el flujo que cambiaste y pega la respuesta real.
- Revisa los logs post-deploy por errores de arranque (pega las líneas relevantes o "0 errores en los primeros N minutos").

### Migración de datos/schema
- Declara en QUÉ entorno corrió. "La migración existe" ≠ "la migración corrió".
- Consulta el schema real después (`\d tabla`, `SHOW CREATE TABLE`) y pega la parte relevante.
- Si migra datos: cuenta antes/después (`SELECT count(*) ...`) y muestrea filas migradas reales.
- La app sigue viva contra el schema nuevo: un request real post-migración pegado.

### UI
- Screenshot o corrida de Playwright del estado REAL renderizado — el JSX correcto no es evidencia de que se ve bien.
- Los estados no felices que tocaste: loading, vacío, error, contenido largo. El que no capturaste, decláralo no verificado.

### Script / job / cron
- Córrelo de verdad, con datos reales o representativos, y pega la salida. Un script nunca ejecutado está roto por defecto (imports, permisos, paths).
- Verifica el efecto en el estado (archivos creados, filas afectadas, mensajes encolados) — no solo el exit code 0.

### Integración externa (API de terceros, webhook, LLM)
- Al menos una llamada REAL (sandbox si existe) con request y respuesta pegados. Los mocks prueban tu código contra tu suposición del contrato, no contra el contrato.
- Si solo pudiste probar con mocks, repórtalo así: "verificado contra mocks; llamada real pendiente".

### Documentación que afirma comportamiento
- Ejecuta cada comando/snippet que la doc afirma, en un entorno limpio si es de instalación. La doc no verificada documenta tu memoria, no el sistema.

## Si no puedes ejecutar la verificación

No inventes un nivel: baja la afirmación. Reporta "aplicado, verificación pendiente", di POR QUÉ no pudiste (sin acceso al entorno, falta credencial) y entrega el comando exacto que el usuario debe correr y qué salida esperar. Ver `references/lenguaje.md`.
