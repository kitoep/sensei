# Componentes — librería primero, nunca reinventar controles

Construir controles interactivos a mano produce bugs de accesibilidad y teclado que no se ven en la demo pero rompen el producto. La regla es librería-primero; lo custom es la excepción justificada.

## Regla dura

**PROHIBIDO construir a mano** cuando existe opción accesible probada en el ecosistema del proyecto:

- Date pickers / time pickers / calendarios
- Dropdowns, selects, comboboxes y autocompletes
- Modales, dialogs, drawers, popovers, tooltips
- Tablas con sort/filtro/paginación/selección
- Tabs, accordions, menús (incluido menú contextual)
- Toasts/notificaciones, sliders, switches

Estos componentes tienen años de casos borde resueltos (foco atrapado en modal, navegación por teclado en combobox, ARIA correcto, scroll-lock, colisiones de popover). Rehacerlos "porque es rápido" descarta ese trabajo y el resultado siempre es inferior.

## Flujo de decisión de librería

1. **¿El proyecto ya usa una librería de componentes?** → Se usa esa. No introduzcas una segunda librería de UI sin discutirlo con el usuario (dos librerías = dos estéticas + doble peso).
2. **¿No hay librería?** → Antes de la primera pantalla, evalúa opciones del ecosistema del framework y **propón al usuario la ideal con justificación** (2–3 candidatas, tu recomendación primero y por qué). La elección es del usuario; el criterio es tuyo.

## Criterios de elección (en este orden)

1. **Accesibilidad incluida**: teclado, foco, ARIA y screen readers resueltos por la librería (no "accesible si tú le agregas"). Es el criterio #1 porque es lo más caro de retrofittear.
2. **Compatibilidad con los tokens del design system**: ¿puedes mapear tu paleta/tipografía/radios sin pelearte con la librería? Prioriza librerías con theming por variables/tokens sobre las de estética cerrada.
3. **Composabilidad**: piezas que se combinan y estilizan, no monolitos config-driven imposibles de ajustar.
4. **Mantenimiento activo**: releases recientes, issues atendidos, comunidad. Una librería muerta es deuda inmediata.
5. **Peso y modelo de entrega**: tree-shakeable / por-componente; en móvil o públicas, el bundle importa.
6. **Compatibilidad real con el framework y su modo de render** (SSR/streaming/nativo según aplique).

## Panorama por tipo (recomendar según el caso, sin casarse con marcas)

- **Headless/unstyled + estilos propios**: primitivos con comportamiento y accesibilidad resueltos, apariencia 100% tuya. Ideal cuando el design system es fuerte y quieres identidad propia. Costo: estilizas todo.
- **Kit completo con theming**: componentes ya estilizados y themables por tokens. Ideal para herramientas internas, admin/dashboards, o cuando la velocidad importa más que la identidad única. Costo: riesgo de verse "de fábrica" (mitigable, abajo).
- **Copy-in / código generado en tu repo**: componentes que viven en tu código y editas libremente. Ideal para productos con diseño propio fuerte y equipo que quiere control total sin mantener primitivos. Costo: las actualizaciones son tuyas.
- **Lo que el proyecto ya tiene** gana por default sobre las tres anteriores.

En móvil nativo o multiplataforma, el equivalente es: componentes del sistema/framework primero, librería del ecosistema después, custom al final.

## Cuándo SÍ construir custom

Solo si se cumple al menos una:

1. **El componente ES el producto** (el editor de un editor, el calendario de una app de agenda, el canvas de una herramienta de diseño): ahí se invierte, con teclado + ARIA + estados completos como parte del alcance, no como mejora futura.
2. **No existe opción accesible probada** para ese patrón en el ecosistema (raro; verifica antes de afirmarlo).
3. Es presentacional puro sin interacción (una card, un badge, un layout): eso no es "control", constrúyelo con los tokens.

## Mapear el design system a la librería (para que no se vea "de fábrica")

1. **Configura el tema de la librería con los tokens** (paleta, tipografía, radios, espaciado) en su mecanismo oficial de theming — nunca sobreescribiendo CSS componente por componente.
2. **Cambia los defaults delatores**: color primario de fábrica, radio default, sombras default y la fuente default SIEMPRE se sustituyen por los del design system. Una librería con sus defaults intactos es el tell #0 de producto genérico.
3. **Crea wrappers delgados** para los 5–8 componentes más usados (Button, Input, Select, Dialog, Toast...) que fijen las variantes permitidas por el design system. Las pantallas consumen los wrappers, no la librería directa — así un cambio de tokens o de librería toca un solo lugar.
4. **Documenta en `docs/design-system.md`** qué librería se eligió, por qué, y la tabla token→variable de tema.
