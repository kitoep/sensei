# sensei 🥋

Español | [English](README.en.md)

**Disciplina de ingeniero senior para Claude Code.**

Son 12 skills que le meten a cualquier modelo la forma de trabajar de un ingeniero con años de experiencia. En vez de dejar que el modelo improvise, lo obligan a frenar antes de actuar, a probar las cosas antes de decir que ya quedaron, a darte una opinión honesta en lugar de darte por tu lado, y a no inventar en las decisiones que salen caras: bases de datos, arquitectura, seguridad y features con IA.

No son consejos sueltos. Cada skill es un protocolo con checklists, plantillas y reglas concretas, hecho para que el modelo no dependa de su criterio ni se saque excusas. Cada uno trae también una tabla con las excusas típicas ("es un cambio chico", "no da tiempo de hacer tests") y la realidad al lado, para cerrarle esas salidas.

## Instalación

Dentro de Claude Code:

```
/plugin marketplace add kitoep/sensei
/plugin install senior-engineer-es@sensei
```

Hay dos versiones, español e inglés. Instala una sola:

- `senior-engineer-es@sensei` para español (skills `senior-engineer-es:*`)
- `senior-engineer-en@sensei` para inglés (skills `senior-engineer-en:*`)

¿Por qué dos versiones y no una que se traduzca sola? Porque cada skill trae tablas de frases prohibidas que atrapan justo lo que el modelo está por escribir. Si programas en inglés tu agente contesta en inglés, así que necesita la versión en inglés para que esas tablas sirvan. En `/plugin` entras a **Discover**, ves las dos y eliges.

El wizard te pregunta el alcance: **User** para todas tus sesiones, o **Project** solo para el proyecto actual. Después corre `/reload-plugins` o reinicia la sesión.

### Un skill a la vez

Si no quieres los 12, cada skill se instala por separado (es la misma copia de archivos; instala la suite completa o los sueltos, no las dos cosas):

```
/plugin install fixer-es@sensei
/plugin install esquema-datos-es@sensei
/plugin install fixer-en@sensei
```

También puedes hacerlo visual: en `/plugin` entras a **Discover** y eliges de la lista. Los skills sueltos funcionan solos; cuando uno menciona a otro, esa referencia es opcional y no truena si el otro no está instalado.

### Actualizar o quitar

```
/plugin marketplace update sensei
/reload-plugins
/plugin uninstall senior-engineer-es@sensei
```

## Los 12 skills

| Skill | Qué le obliga a hacer al modelo |
|---|---|
| `planning` | Antes de escribir un plan hace las preguntas que hacen falta (las cinco capas) y revisa qué ya existe en el proyecto, para no volver a construir algo que ya estaba. Parte el trabajo en tareas que se pueden verificar y las ordena por riesgo. |
| `architect` | No te recomienda stack sin antes preguntarte las nueve cosas que de verdad definen la decisión. Baja a tierra las fantasías de escala y cada elección la justifica con una condición real tuya. Si no la puede justificar, es moda, y lo dice. |
| `project-bootstrap` | Nada de código hasta terminar el intake. Las decisiones que luego salen carísimas de cambiar (multi-tenancy, auth, borrado de datos) se toman desde el día uno. Primero un esqueleto que corre de punta a punta, luego las features. |
| `designer` | Regla que no se salta: no maqueta ni una pantalla hasta que exista un design system aprobado por ti. Trae el catálogo de las 12 señales que delatan un diseño hecho por IA, exige los 4 estados de cada pantalla (cargando, vacío, error, con datos) y corre antes que cualquier otro skill de diseño o de estética. |
| `critic` | Te critica de frente, sin adularte. Arranca con el veredicto en la primera línea, primero arma la versión más fuerte de tu idea y luego la ataca, y te da mínimo 3 riesgos y 1 alternativa. Si lo presionas, no se echa para atrás. |
| `fixer` | Para arreglar un bug, primero lo reproduce; sin reproducción no hay arreglo. Plantea hipótesis que se puedan descartar, va por la causa de raíz y busca el mismo patrón en todo el repo. Al test que prueba el arreglo lo ve fallar antes de confiar en él. Y trae un protocolo para cuando algo "sigue fallando". |
| `tester` | Arma el plan de pruebas desde lo que no puede romperse, empezando por los casos que rompen y dejando el camino feliz al final. Siempre incluye la UI. No da nada por probado si quedan casos importantes sin correr. |
| `security-compliance` | Dos modos: auditar código en busca de huecos, y avisarte mientras desarrollas. Revisa OWASP con búsquedas concretas, aislamiento entre clientes (multi-tenant y RLS) y protección de datos (GDPR, PCI y la ley mexicana). Solo marca algo como grave si arma el escenario que lo demuestra. |
| `verificador` | Nada se declara terminado sin la prueba de que corre, pegada en el mismo mensaje. Distingue entre "verificado" y "lo apliqué pero no lo probé", y no reparte la palomita ✅ de a gratis. |
| `esquema-datos` | Bases de datos y migraciones que no tumban producción. Pregunta por las consultas y el volumen antes de diseñar, nunca usa float para el dinero, hace los cambios peligrosos en dos pasos (expand/contract), pone los candados en la base de datos y comprueba los índices con EXPLAIN. |
| `features-ia` | Features con LLMs sin inventar. Tiene prohibido escribir nombres de modelo o parámetros de memoria (los verifica contra la documentación), mete todo detrás de una sola capa que no depende del proveedor, arma las tools con contratos estrictos y filtrando por usuario, pone defensas contra prompt injection y no deja cambiar un prompt sin correr su set de pruebas. |
| `entrega-git` | Commits de una sola cosa, con el diff revisado archivo por archivo (nunca `git add .` a ciegas). Los mensajes explican el porqué, no el qué. Los PR tratan un solo tema y traen una descripción que se puede auditar. Nada destructivo sin que se lo pidas. |

## Cómo están hechos

- Cada skill tiene un `SKILL.md` corto (el protocolo, las reglas, las señales de alerta) y una carpeta `references/` con el detalle, que el modelo abre solo cuando lo necesita. Mientras no se usa, casi no ocupa contexto.
- El contenido está en español y las descripciones en inglés (así las reconoce Claude Code). Las tablas de frases prohibidas van en español a propósito, porque atrapan justo lo que el modelo está por escribir.
- La frase que cruza todos: **"Violar la letra de estas reglas es violar su espíritu."** No hay tarea "demasiado simple" para saltarse el proceso.
- Los 12 se probaron con un método sencillo: el mismo pedido, una vez sin el skill y otra con él, sobre proyectos de prueba con trampas puestas a propósito. En los 12 el skill cambió la conducta del modelo para bien. Desde exigir pruebas donde antes había un "listo ✅" vacío, hasta frenar una migración que tumbaba producción y tapar una fuga de datos entre clientes en un bot.

## Instalación manual (sin plugin)

Copia las carpetas de `senior-engineer-es/skills/` (o las de `senior-engineer-en/skills/`) a tu carpeta de skills (`~/.claude/skills/`) y reinicia la sesión. Así pierdes las actualizaciones automáticas del marketplace.

## Licencia

MIT. [kitoep](https://github.com/kitoep)
