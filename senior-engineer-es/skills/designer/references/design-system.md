# Design system primero — cómo construirlo CON el usuario

El design system se construye en diálogo con el usuario ANTES de la primera pantalla. No es un documento burocrático: son las decisiones visuales tomadas una sola vez para no re-decidirlas (mal) en cada pantalla. Salida final: un archivo en el proyecto (por defecto `docs/design-system.md`) que toda pantalla futura respeta como contrato.

## Fase 1 — Intake de identidad (preguntas al usuario)

Haz estas preguntas de una en una o agrupadas en un solo mensaje con opciones; no avances a tokens sin respuestas. Las marcadas ⭐ son las 4 esenciales de la versión rápida.

1. ⭐ **¿Quién es el usuario final y en qué contexto usa esto?** — ¿Con prisa o con calma? ¿Móvil o escritorio? ¿Experto que lo usa 8 horas al día o novato ocasional? Esto decide densidad, tamaño de tipografía y tolerancia a la curva de aprendizaje. Un dashboard para operadores expertos es denso y sobrio; un onboarding para novatos es aireado y guiado.
2. ⭐ **¿Qué 3 adjetivos debe transmitir el producto?** — Pide exactamente 3 (p. ej. "confiable, moderno, cálido"). Cada decisión visual posterior se valida contra ellos: ¿este color/tipografía/espaciado transmite esos adjetivos?
3. ⭐ **¿Referencias que te gustan y POR QUÉ?** — Pide 1–3 productos/sitios que le gusten al usuario y qué le gusta de cada uno (¿la densidad? ¿el color? ¿la tipografía?). El "por qué" es lo valioso; la referencia sin por qué produce imitación superficial.
4. ⭐ **¿Marca existente o desde cero?** — Si hay logo/colores de marca, se heredan como restricción. Si no, propondrás paleta desde los adjetivos.
5. **¿Modo oscuro?** — Se decide AHORA, no después: agregar dark mode a posteriori duplica el costo. Si la respuesta es "algún día", define los tokens como variables semánticas desde el inicio (`--surface`, `--text-primary`) aunque solo se implemente el modo claro.
6. **¿Idioma(s) del contenido y tono?** — Afecta longitudes de texto (alemán rompe layouts que el inglés no) y microcopy.

## Fase 2 — Direcciones visuales (ANTES de tokenizar)

Con el intake, presenta al usuario **2–3 direcciones visuales distintas** para que elija. Cada dirección se describe en 4–6 líneas: personalidad, paleta tentativa (nombra los colores), tipografía tentativa, densidad, un elemento distintivo. Ejemplo de formato:

> **Dirección A — "Clínica de confianza":** neutros cálidos + un verde profundo como primario; tipografía serif para títulos (autoridad) y sans para UI; espaciado generoso; distintivo: tarjetas con borde fino en lugar de sombra.
> **Dirección B — "Herramienta pro":** fondo gris frío casi blanco, primario azul saturado usado con MUCHA moderación; una sola sans en 2 pesos; denso; distintivo: tipografía tabular para números.

Si el medio lo permite, materializa las direcciones en una pantalla de muestra HTML estática para que el usuario VEA en lugar de imaginar. El usuario elige (o mezcla); solo entonces tokenizas.

## Fase 3 — Tokens concretos

Documenta TODOS estos grupos con valores exactos. Un token no definido = una decisión que se tomará mal a las prisas después.

### Color
- **Primario**: 1 color, con su escala de uso (hover, pressed, subtle/bg). Regla: el primario aparece en ≤10% de la superficie de una pantalla típica; su función es señalar la acción principal, no decorar.
- **Neutros**: escala de 5–8 pasos (fondo, superficie, borde, texto secundario, texto primario) elegidos con intención — decide si son grises fríos, cálidos o puros según los adjetivos.
- **Semánticos**: éxito, error, warning, info. Nunca se usa el semántico para decoración ni el primario para estados.
- **Verificación AA obligatoria**: todo par texto/fondo definido debe cumplir WCAG AA (4.5:1 texto normal, 3:1 texto grande y componentes UI). Verifica los ratios calculándolos, no "a ojo"; documenta el ratio junto al par.

### Tipografía
- Máximo 2 familias (1 es válido y suele ser mejor). Anota el fallback stack.
- **Escala de tamaños fija** (no tamaños ad-hoc): define 6–8 pasos con nombre y uso, p. ej. `xs 12 / sm 14 / base 16 / lg 18 / xl 22 / 2xl 28 / 3xl 36` con line-height por paso. Toda pantalla usa SOLO estos pasos.
- Pesos permitidos (típicamente 2–3: regular, medium, bold) y dónde se usa cada uno.
- Números en datos/tablas: usa variante tabular (`font-variant-numeric: tabular-nums`) si el producto muestra cifras.

### Espaciado, radios, sombras, bordes
- **Escala de espaciado fija en base 4 u 8**: p. ej. `4, 8, 12, 16, 24, 32, 48, 64`. Prohibido cualquier margen/padding fuera de la escala.
- **Radios**: define 2–3 (p. ej. `sm 4, md 8, full`) y a qué se aplica cada uno. Radio uniforme gigante en todo = tell de diseño genérico.
- **Sombras**: define 2–3 niveles de elevación con valores exactos y cuándo se usa cada uno (o decide "este producto usa bordes, no sombras" — es una dirección válida y distintiva).
- **Bordes**: grosor y color estándar.

### Modo oscuro (si aplica)
- Tokens semánticos con valor por modo. Regla: en oscuro no se invierten colores, se re-eligen (los neutros oscuros llevan menos contraste entre pasos; las sombras se sustituyen por diferencia de superficie).

## Fase 4 — Documentar y cerrar

1. Escribe `docs/design-system.md` con: resumen del intake (target, adjetivos, dirección elegida), todos los tokens con valores, y las reglas de uso (primario ≤10%, escala de espaciado obligatoria, pares AA verificados).
2. Si el proyecto usa un sistema de estilos (CSS variables, Tailwind config, tema de la librería), **materializa los tokens ahí en el mismo PR** — el doc y el código nunca divergen.
3. Pide aprobación explícita del usuario sobre el documento. Solo con aprobación se abre la puerta a escribir pantallas.

## Gobernanza posterior

- Una pantalla nueva que "necesita" un color/tamaño/espacio fuera de tokens → primero se discute agregar el token (con el usuario), después se usa. Nunca un valor mágico inline.
- Cambios al design system se hacen en su archivo + su materialización en código, nunca sobreescribiendo estilos localmente en una pantalla.
