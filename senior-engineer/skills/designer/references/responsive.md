# Responsive — cuando aplica, con targets reales

"Responsive" no es un default que se aplica a ciegas: es una decisión de producto. El error genérico es doble: hacer responsive lo que nunca se usará en móvil (costo sin valor) o "hacer que se encoja" lo que sí (móvil de segunda). Primero targets, luego patrones.

## Fase 0 — Decidir targets con el usuario

Pregunta ANTES de maquetar la primera pantalla (idealmente en el intake del design system):

1. **¿Dónde se usará esto DE VERDAD?** — No "¿lo quieres responsive?" (todos dicen sí), sino: "¿tu usuario final va a operar esta pantalla desde el teléfono? ¿en qué situación?". Un panel de administración que se usa en oficina es escritorio-primero; un portal donde el cliente final agenda/consulta es móvil-primero casi siempre.
2. **¿Hay pantallas con targets distintos?** — Es normal: el admin en escritorio, la vista del cliente en móvil. Documenta el target POR ÁREA, no uno global.
3. **Registra la decisión en `docs/design-system.md`**: targets por área + breakpoints del proyecto.

Regla de estrategia: **mobile-first solo si móvil es target real** de esa área. Si el target es escritorio, diseña escritorio-primero y define qué pasa en ventanas chicas (mínimo: nada roto ni ilegible, scroll horizontal solo en tablas).

## Breakpoints

- Se definen UNA vez en el design system (típicamente 3–4: p. ej. `sm 640 / md 768 / lg 1024 / xl 1280` o los del framework de estilos del proyecto) y se usan SIEMPRE por nombre. Prohibido inventar breakpoints ad-hoc por pantalla.
- Los breakpoints cortan donde el CONTENIDO lo pide, no por lista de dispositivos. Si un layout se rompe a 900px, el ajuste va en el breakpoint estándar más cercano, no en uno nuevo de 900.

## Patrones de adaptación (en vez de "que se encoja")

Cada tipo de layout tiene su transformación. Encoger proporcionalmente NO es adaptar:

| Escritorio | Móvil | Nota |
|---|---|---|
| Tabla de datos multicolumna | Cards apiladas con los 2–3 campos clave + detalle al tocar | Elegir QUÉ campos sobreviven es decisión de diseño; pregúntate qué busca el usuario en la fila |
| Sidebar de navegación | Drawer (hamburguesa) o tab bar inferior si son ≤5 destinos frecuentes | Tab bar > drawer para navegación primaria de uso constante |
| Grid de N columnas | Stack de 1 columna (o 2 si los items son compactos) | El ORDEN del stack se decide: lo dominante primero, no el orden del DOM por accidente |
| Modal centrado grande | Sheet de pantalla completa o bottom-sheet | Un modal de escritorio encogido en móvil es inusable |
| Formulario de 2 columnas | 1 columna siempre | Formularios multicolumna en móvil generan errores de captura |
| Acciones en hover (iconos que aparecen) | Acciones siempre visibles o tras menú explícito | En táctil no existe hover: cualquier funcionalidad solo-hover está ROTA en móvil |
| Tooltip con información necesaria | La información va visible o tras tap explícito | Mismo motivo |

## Reglas táctiles y de legibilidad

1. **Targets táctiles ≥44×44px** (o el equivalente de la plataforma) en todo elemento interactivo móvil, con separación suficiente para no tocar el vecino.
2. **Tipografía base en móvil ≥16px** en inputs y cuerpo (evita zoom automático del navegador en iOS y es el piso de legibilidad).
3. **Anchos de lectura**: en pantallas grandes, el texto corrido se limita (~65–75 caracteres por línea); una landing que estira párrafos a 1400px de ancho está rota en la dirección opuesta.
4. **Zonas del pulgar**: en móvil, la acción primaria de la pantalla vive en la mitad inferior alcanzable; lo destructivo lejos de las zonas de toque fácil.

## Verificación — parte del done de cada pantalla

Ninguna pantalla con target multi-dispositivo se declara terminada sin verificarse en viewports concretos:

- [ ] Render en los viewports acordados (mínimo: uno móvil ~375px, uno escritorio ~1440px; tablet si es target). Usa las herramientas del proyecto (navegador con device toolbar, Playwright con viewports, o captura) — verificación observada, no "debería verse bien".
- [ ] Cero scroll horizontal accidental en móvil (el scroll horizontal solo existe donde se diseñó, p. ej. dentro de una tabla).
- [ ] Todo lo interactivo alcanzable y ≥44px en móvil; nada depende de hover.
- [ ] Textos reales largos no rompen el layout (nombre largo, email largo, cifra grande).
- [ ] Los 4 estados (cargando/vacío/error/datos) también se ven correctos en móvil, no solo el estado feliz de escritorio.
