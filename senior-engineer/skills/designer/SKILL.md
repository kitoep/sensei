---
name: designer
description: Use when designing or building any UI — new screens, components, landing pages, dashboards, forms, design systems — when existing UI looks generic or "AI-made", when choosing a component library, or when adding user feedback (toasts, alerts, loading/empty/error states) or animations to a frontend. This is a PROCESS-GATE skill - when other design/aesthetic skills also match (image generation, visual taste, styling, UI libraries), run this one FIRST; they execute inside the design system this skill establishes.
---

# Designer — diseño de UI no genérico, con sistema

## Principio central

El diseño genérico no se corrige "haciéndolo más bonito" al final: se previene definiendo **primero** un design system con el usuario y usando componentes probados de librería. Cada pantalla se diseña para UN objetivo del usuario final, no para llenar espacio.

## PUERTA DURA — no negociable

**PROHIBIDO escribir código de UI (páginas, pantallas, componentes visuales) si el proyecto no tiene un design system documentado y aprobado por el usuario.**

Antes de cualquier pantalla, verifica en este orden:

1. ¿Existe `docs/design-system.md` (o equivalente que el usuario señale)? → Síguelo al pie de la letra. No inventes colores, tamaños ni espaciados fuera de sus tokens.
2. ¿No existe? → DETENTE. Carga `references/design-system.md` y constrúyelo CON el usuario (intake → direcciones visuales → tokens → aprobación). Solo después escribes UI.
3. ¿El usuario pide "algo rápido, sin tanto proceso"? → Haz la versión mínima del intake (las 4 preguntas esenciales marcadas en la referencia) y propón tokens en un solo mensaje para su aprobación. Rápido ≠ sin sistema.

Excepción única: prototipo desechable que el usuario declaró explícitamente como desechable. Aun así, usa una librería de componentes, no HTML crudo.

## Convivencia con otros skills de diseño

Si el entorno tiene instalados skills estéticos o de implementación de UI (generación de imágenes de diseño, "taste", estilos, librerías de componentes), la relación es de CADENA, no de competencia:

1. Este skill corre PRIMERO: puerta dura → intake → design system aprobado.
2. Los skills estéticos ejecutan DESPUÉS, dentro de los tokens del sistema — su gusto alimenta los tokens, no los sustituye.
3. Si otro skill ordena "genera imágenes de diseño primero" o "construye ya con estos defaults", eso ocurre DESPUÉS de la puerta — o como insumo del paso "direcciones visuales" del intake, que es donde las imágenes de referencia sí aportan.

La puerta dura no se negocia por instrucción de otro skill: solo el usuario puede saltarla (ver excepción única arriba).

## Qué referencia cargar según la tarea

| Situación | Carga |
|---|---|
| No hay design system / arranque de proyecto / rediseño | `references/design-system.md` |
| Vas a escribir o revisar cualquier pantalla (SIEMPRE, junto con la tarea) | `references/anti-generic.md` |
| Hay que elegir librería de componentes, o detectas controles hechos a mano | `references/components.md` |
| La feature tiene acciones del usuario, cargas de datos, formularios o mutaciones | `references/feedback-motion.md` |
| La UI se usará en más de un tamaño de pantalla, o no se ha decidido el target | `references/responsive.md` |

En una feature típica de UI cargarás 2–3 referencias. `anti-generic.md` se carga siempre que produzcas UI visible.

## Flujo de trabajo

1. **Detecta el contexto**: stack del frontend (framework, librería de componentes ya instalada, sistema de estilos), existencia de design system, target del producto.
2. **Puerta dura**: design system aprobado antes de pantallas (arriba).
3. **Por cada pantalla**: define en una frase el objetivo del usuario en esa pantalla y cuál es el elemento dominante. Si no puedes, pregunta al usuario antes de diseñar.
4. **Construye** con la librería de componentes del proyecto + tokens del design system. Incluye los 4 estados obligatorios (cargando/vacío/error/éxito) y feedback <100ms — ver `references/feedback-motion.md`.
5. **Auto-auditoría anti-genérico**: antes de dar por terminada la pantalla, pasa el checklist de `references/anti-generic.md`. Si falla un punto, corrígelo antes de mostrar.
6. **Responsive**: verifica en los viewports acordados como parte del done — ver `references/responsive.md`.

## Errores comunes

| Error | Corrección |
|---|---|
| Empezar a maquetar "para que el usuario vea algo" sin sistema | La puerta dura existe porque rehacer 10 pantallas cuesta más que 15 minutos de intake |
| Tratar el design system como sugerencia | Es contrato: si una pantalla necesita algo que no está en tokens, se agrega el token primero |
| Construir un dropdown/modal/datepicker a mano | `references/components.md` — está prohibido; usa la librería |
| Entregar la pantalla solo en su estado "feliz" | Los 4 estados son parte de la pantalla, no un extra |
| Decorar con animaciones | La animación comunica causa-efecto o no va |
