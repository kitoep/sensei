# sensei 🥋

Español | **[English](README.en.md)**

**Disciplina de ingeniero senior para Claude Code.**

12 skills para Claude Code que le instalan a cualquier modelo la disciplina de un ingeniero senior: puertas duras antes de actuar, evidencia ejecutada antes de declarar nada terminado, crítica honesta sin adulación, y cero improvisación en las decisiones caras (schemas, arquitectura, seguridad, features con IA).

No son "tips": son protocolos prescriptivos — checklists, plantillas y reglas exactas — escritos para que el modelo no dependa de su propio criterio ni pueda racionalizar excepciones. Cada skill incluye la tabla de racionalizaciones conocidas ("es un cambio trivial", "no hay tiempo para tests"...) con su realidad al lado, para cerrarle las salidas.

## Instalación

Dentro de Claude Code:

```
/plugin marketplace add kitoep/sensei
/plugin install senior-engineer-es@sensei        # español
```

El paquete viene en **dos idiomas** — instala uno u otro (no ambos):

- `senior-engineer-es@sensei` — **español** (skills `senior-engineer-es:*`)
- `senior-engineer-en@sensei` — **English** (skills `senior-engineer-en:*`)

¿Por qué dos versiones y no una traducida al vuelo? Porque las tablas de "frases prohibidas" de cada skill cazan literalmente lo que el modelo está a punto de escribir; si programas en inglés, tu agente responde en inglés y necesita la versión en inglés para que esas tablas funcionen. En `/plugin` → **Discover** ves ambas y eliges.

El wizard pregunta el scope: **User** (todas tus sesiones) o **Project** (solo el proyecto actual). Luego `/reload-plugins` o reinicia la sesión.

### À la carte

¿No quieres los 12? Cada skill también es instalable por separado (misma copia de archivos; elige suite **o** individuales, no ambos):

```
/plugin install fixer-es@sensei              # español
/plugin install esquema-datos-es@sensei
/plugin install fixer-en@sensei           # English (sufijo -en)
```

O visual: `/plugin` → pestaña **Discover** → elige de la lista. Los skills individuales funcionan solos — sus referencias cruzadas son condicionales y no dependen de que el resto esté instalado.

### Actualizar / desinstalar

```
/plugin marketplace update sensei      # trae la última versión del repo
/reload-plugins
/plugin uninstall senior-engineer-es@sensei
```

## Los 12 skills

| Skill | Qué le impone al modelo |
|---|---|
| `planning` | Las 5 capas de preguntas antes de escribir un plan; reconciliar contra el estado real del proyecto ANTES de planear (no re-construir lo que ya existe); descomposición en tareas verificables ordenadas por riesgo. |
| `architect` | Prohibido recomendar stack sin intake de 9 preguntas sobre las condiciones reales; desinfla fantasías de escala; cada elección se traza a una condición ("si no se traza, es moda"). |
| `project-bootstrap` | Cero código antes del intake; los cimientos carísimos de retrofitear (multi-tenancy, auth, borrado) se deciden el día 1; esqueleto andante antes que features. |
| `designer` | Puerta dura: no se escribe UI sin design system aprobado; catálogo de los 12 "tells" del diseño hecho por IA; 4 estados obligatorios por pantalla; actúa como puerta de proceso ANTES de cualquier skill estético. |
| `critic` | Crítica sin adulación: la primera oración es el veredicto; steelman → suposición de carga → mínimo 3 riesgos + 1 alternativa; mantener posición bajo presión. |
| `fixer` | Debugging con puertas: sin reproducción no hay fix; hipótesis falsables; causa raíz y patrón simétrico en todo el repo; el test de regresión se ve FALLAR antes de creerle; protocolo "sigue fallando". |
| `tester` | Plan de pruebas desde invariantes (adversarial primero, happy path al final); la UI siempre se incluye; nada se declara probado con casos P1-P2 pendientes. |
| `security-compliance` | Auditoría (AUDIT) y prevención (GUARD): OWASP con patrones grep, aislamiento multi-tenant/RLS, protección de datos (GDPR/LFPDPPP/PCI), severidad solo con escenario verificado, tests adversariales. |
| `verificador` | Anti-"falso verde": nada se declara terminado sin evidencia ejecutada y pegada; escalera de evidencia por tipo de tarea; "verificado" ≠ "aplicado, sin verificar"; el ✅ se gana, no se regala. |
| `esquema-datos` | Schemas y migraciones que no rompen producción: intake de consultas/volumen antes de diseñar; nunca float para dinero; expand/contract para todo cambio incompatible; constraints en la DB como garantía final; índices verificados con EXPLAIN. |
| `features-ia` | Features con LLMs sin inventar: prohibido escribir nombres de modelo/params de memoria (verificar contra docs); capa única agnóstica de proveedor; tool-calling con contratos estrictos y scoping por usuario; guardrails contra injection; golden set de evals como puerta para cambiar prompts. |
| `entrega-git` | Commits atómicos con el diff revisado archivo por archivo (nunca `git add .` a ciegas); mensajes con el porqué; PRs de un tema con descripción auditable; operaciones destructivas prohibidas sin pedido explícito. |

## Cómo están construidos

- Cada skill es un `SKILL.md` ligero (el protocolo, las puertas, las red flags) + `references/` con el conocimiento detallado que el modelo carga solo cuando lo necesita — el costo en contexto es mínimo hasta que el skill se activa.
- Contenido en **español**, descriptions en inglés (para el matching de Claude Code). Las tablas de frases prohibidas están en español a propósito: cazan literalmente lo que el modelo está a punto de escribir.
- Doctrina firma: **"Violar la letra de estas reglas es violar su espíritu"** — no hay tareas "demasiado simples" para el proceso.
- Los 12 fueron validados con pruebas RED/GREEN (mismo prompt neutro, con y sin el skill, sobre sandboxes ejecutables con trampas plantadas): 12/12 cambiaron la conducta del modelo en la dirección correcta — desde imponer evidencia donde había un "listo ✅" vacío, hasta impedir una migración que rompía producción y una fuga de datos entre clientes en un bot.

## Instalación manual (sin plugin)

Copia las carpetas de `senior-engineer-es/skills/` (o `senior-engineer-en/skills/`) a tu directorio personal de skills (`~/.claude/skills/`) y reinicia la sesión. Pierdes las actualizaciones automáticas del marketplace.

## Licencia

MIT — [kitoep](https://github.com/kitoep)
