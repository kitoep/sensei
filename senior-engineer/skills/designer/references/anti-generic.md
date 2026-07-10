# Anti-genérico — catálogo de "tells" de diseño IA y reglas correctivas

Un diseño genérico no es feo: es intercambiable. Se ve igual que mil productos y no transmite los adjetivos del design system. Este archivo se carga SIEMPRE que se produce UI visible. Úsalo en dos momentos: al diseñar (evitar) y antes de entregar (auto-auditoría).

## Catálogo de tells (si aparece uno, corrígelo)

| # | Tell "lo hizo una IA" | Regla correctiva |
|---|---|---|
| 1 | Gradiente morado/azul (o cualquier gradiente) como decoración por default | Color plano del design system. Gradiente solo si la dirección visual elegida lo define explícitamente como distintivo |
| 2 | Hero centrado: título grande + subtítulo + 2 botones + 3 cards de features abajo | Rompe la simetría: layout asimétrico, contenido real del producto (screenshot, dato vivo, demo) en lugar de cards de relleno |
| 3 | Emojis como iconos (🚀 ✨ 💡 en features/títulos) | Set de iconos consistente de UNA librería (la que use el proyecto). Cero emojis en UI |
| 4 | Sombras grandes y difusas en todas las cards | Elevación según el design system: máximo 2–3 niveles; considera bordes finos en vez de sombra |
| 5 | border-radius gigante y uniforme en todo (botones, cards, inputs, modales) | Radios del design system por tipo de elemento; los contenedores grandes llevan radio menor o proporcional |
| 6 | Copy placeholder: "Unlock the power of...", "Lleva tu X al siguiente nivel", lorem ipsum | Microcopy real del dominio del producto, escrito para el target del intake. Si no conoces el contenido real, pregunta — el copy ES diseño |
| 7 | 5 tonos de gris sin intención mezclados | Solo los neutros de la escala del design system |
| 8 | Todo centrado (títulos, párrafos, listas, formularios) | Alineado a la izquierda por default en contenido y formularios. Centrado solo en momentos ceremoniales cortos (estado vacío, confirmación) |
| 9 | Densidad uniforme: todo aireado o todo apretado, sin jerarquía | Densidad según target (ver abajo) y VARIADA: lo importante respira, lo secundario se compacta |
| 10 | Primario regado por toda la pantalla (botones, links, iconos, bordes, badges) | Primario ≤10% de la superficie; una pantalla típica tiene UN elemento primario dominante |
| 11 | Cards dentro de cards dentro de cards | Máximo un nivel de contenedor con fondo/borde; agrupa con espacio y tipografía, no con más cajas |
| 12 | Tabla/lista de datos con el mismo peso visual en todas las columnas | Jerarquiza: la columna que el usuario busca va primero y con mayor peso; metadatos en texto secundario |

## Reglas de construcción (positivas)

1. **Una decisión distintiva y consistente > diez efectos.** Elige UN elemento memorable del design system (una tipografía con carácter, un uso de color inusual, un estilo de borde) y aplícalo con disciplina en todo el producto. Eso es identidad; los efectos acumulados son ruido.
2. **La jerarquía se construye con tamaño, peso y espacio — no con más colores.** Antes de agregar un color para destacar algo, prueba: más tamaño, más peso, más espacio alrededor. El color es el último recurso y sale del design system.
3. **Cada pantalla responde "¿qué quiero que el usuario haga aquí?" con UN elemento dominante.** Escríbelo en una frase antes de maquetar. Dos llamadas a la acción con el mismo peso = ninguna decisión tomada. Todo lo demás en la pantalla se subordina visualmente a ese elemento.
4. **Densidad según target** (del intake): herramienta de uso intensivo/experto → densa, tipografía base 14, más filas visibles; producto de consumo/ocasional → aireada, base 16, una idea por vista. Nunca "densidad default".
5. **El contenido real manda.** Diseña con datos reales o realistas del dominio (nombres, cifras, textos largos y cortos). El diseño que solo funciona con "Juan Pérez — Lorem ipsum" está roto.

## Auto-auditoría antes de entregar (checklist obligatorio)

Pasa la pantalla por estas preguntas; si alguna falla, corrige antes de mostrar al usuario:

- [ ] ¿Cero tells de la tabla (recorre los 12)?
- [ ] ¿Puedo nombrar el elemento dominante de la pantalla y es visualmente obvio?
- [ ] ¿Todos los valores (color, tamaño, espacio, radio) vienen de tokens del design system?
- [ ] ¿El copy es real y del dominio (cero placeholder)?
- [ ] ¿Transmite los 3 adjetivos del intake? (nómbralos y justifica en una línea)
- [ ] ¿Un usuario podría distinguir este producto de un template genérico en una captura sin logo?
