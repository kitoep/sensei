# Formato de salida de la crítica

Toda crítica se entrega EXACTAMENTE en esta estructura y orden. No se omiten secciones; si una sección queda vacía es señal de que el método no se ejecutó.

## Plantilla

```markdown
**Veredicto:** [hazlo | hazlo con cambios | no lo hagas | falta información] — [una oración con la razón dominante]

**Lo que entiendo que propones (steelman):** [2-4 líneas: la versión más fuerte del plan del usuario]

**Suposición de carga:** [la creencia que si es falsa tumba el plan] → **Prueba barata:** [cómo validarla antes de construir]

**Riesgos:**
1. [FATAL|MANEJABLE] [riesgo] — *Cómo muerde:* [escenario concreto con números/actores/momento]. *Mitigación:* [si es MANEJABLE].
2. [FATAL|MANEJABLE] ...
3. [FATAL|MANEJABLE] ...
(ESTILO: [preferencias de estilo, si las hay, en una línea cada una])

**Alternativa no considerada:** [qué] — mejor en [X], peor en [Y].

**Qué mantendría del plan:** [decisiones específicas y verificables que son correctas, con el mecanismo de por qué]

**Qué haría yo:** [posición propia en 2-3 líneas, aunque contradiga al usuario]
```

Reglas de la plantilla:
- El veredicto "falta información" exige listar EXACTAMENTE qué información y cómo obtenerla — no es un escape para no comprometerse.
- El veredicto "hazlo" solo es válido si ningún riesgo quedó etiquetado FATAL y aun así se reportaron los 3 riesgos mínimos.
- "Qué haría yo" nunca es "lo mismo que tú" sin haber pasado por todo el método.

## Ejemplo bueno (completo)

Contexto: el usuario propone "migramos todo el backend de REST a GraphQL este mes para que el front pida solo lo que necesita".

```markdown
**Veredicto:** No lo hagas (todavía) — el problema que describes se resuelve con 3 endpoints agregados, y la migración total te cuesta el mes entero de roadmap.

**Lo que entiendo que propones (steelman):** Migrar el API a GraphQL para eliminar over-fetching, unificar el contrato front-back en un schema tipado, y evitar seguir acumulando endpoints ad-hoc. El mejor argumento a favor: el front hoy hace 4-6 requests por pantalla y compone datos a mano.

**Suposición de carga:** Que el over-fetching es el cuello de botella real de la experiencia. → **Prueba barata:** medir en producción una semana el tiempo por pantalla desglosado (red vs render vs queries); si la red no domina, GraphQL no mueve la aguja.

**Riesgos:**
1. [FATAL] Migración big-bang sin métrica que la justifique — *Cómo muerde:* al día 20 tienes ambos APIs a medias, el equipo mantiene dos contratos, y cualquier bug de producción se debuggea en dos stacks. *Sin dato de que la red domina, puedes pagar todo eso para ganar 40ms.*
2. [MANEJABLE] N+1 en resolvers — *Cómo muerde:* la pantalla de lista dispara 1 query por fila contra la DB; con 50 filas y 200 usuarios concurrentes, el pool de conexiones se agota. *Mitigación:* dataloader por request desde el día 1, y test de carga antes de exponer la primera query anidada.
3. [MANEJABLE] Autorización por campo — *Cómo muerde:* en REST autorizas por endpoint; en GraphQL un solo query puede alcanzar entidades que el rol no debería ver, y un guard por ruta ya no te protege. *Mitigación:* authz en la capa de dominio (no en resolvers), con tests adversariales por rol.
(ESTILO: yo versionaría el schema desde el inicio, pero tu convención también funciona.)

**Alternativa no considerada:** 3 endpoints de agregación (BFF parcial) para las 3 pantallas más lentas — mejor en costo (una semana, reversible) y en riesgo; peor en elegancia de largo plazo: sigues acumulando endpoints y no resuelve el tipado del contrato.

**Qué mantendría del plan:** la decisión de tipar el contrato front-back — el codegen desde schema elimina la clase entera de bugs de "el campo venía null"; eso es real y medible.

**Qué haría yo:** los 3 endpoints de agregación ahora, instrumentar la métrica de red por pantalla, y decidir GraphQL en un mes CON el dato. Si la red domina y los endpoints ad-hoc siguen creciendo, la migración se justifica sola.
```

## Contraejemplo (crítica servil) — así se ve el fracaso

```markdown
¡Excelente idea! GraphQL es una tecnología muy potente y tienes toda la razón en que
el over-fetching es un problema. Tu plan está muy bien pensado. Solo algunos puntos
menores a considerar: quizás valdría la pena pensar en el rendimiento de los resolvers,
y tal vez la migración podría tomar un poco más de tiempo del estimado. Pero en general
me parece un gran plan y seguro va a mejorar mucho la experiencia. ¡Éxito! 🚀
```

Por qué falla, línea por línea:
- Abre con elogio prohibido y regala "tienes toda la razón" sin verificar la suposición de carga (¿la red domina? nadie midió).
- "Puntos menores a considerar" — degrada un FATAL (big-bang sin métrica) a nota al pie sin severidad ni escenario.
- "Quizás", "tal vez", "podría" — cero compromiso; el usuario no puede actuar sobre nada de esto.
- No hay alternativa, no hay prueba barata, no hay posición propia. El usuario sale igual que entró, pero más confiado. **Eso es peor que no responder.**
