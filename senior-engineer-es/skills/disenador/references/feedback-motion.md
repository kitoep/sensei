# Feedback y motion — el usuario nunca se queda sin respuesta

Una UI se siente rota cuando el usuario actúa y no pasa nada visible. Regla madre: **toda acción del usuario tiene respuesta visible en <100ms** — aunque la operación real tarde segundos, el acuse es inmediato (estado pressed, spinner en el botón, optimistic update). Congelar la UI o dejar el botón igual mientras "piensa" está prohibido.

## Qué patrón usar en cada caso (tabla de decisión)

| Situación | Patrón obligatorio | Nunca |
|---|---|---|
| Confirmación de acción exitosa no bloqueante (guardado, enviado, copiado) | Toast breve (3–5s), auto-desaparece, con undo si la acción es reversible | Modal de "¡Éxito!" que hay que cerrar |
| Error de validación de un campo | Mensaje inline junto al campo, en el submit o blur; el campo marca estado de error; foco al primer campo con error | Toast o alert genérico "revisa el formulario" |
| Error de operación (red, servidor) | Mensaje en el contexto de la acción con botón de reintento; conserva lo que el usuario escribió | Perder el input del usuario; error solo en consola |
| Decisión destructiva o irreversible (borrar, cancelar suscripción) | Modal de confirmación que nombra el objeto ("¿Eliminar 'Consulta del 12 mar'?") y botón destructivo diferenciado | Confirmar con toast, o borrar sin confirmación |
| Carga inicial de página/vista | Skeleton que respeta el layout final (no salta al cargar) | Spinner de página completa; pantalla en blanco |
| Acción corta (<1s esperado) disparada por botón | Spinner/estado loading DENTRO del botón, botón deshabilitado mientras | Deshabilitar sin indicador; spinner global |
| Operación larga (>3s) | Progreso o mensaje de estado + posibilidad de seguir usando la app si es posible | Bloquear todo sin información |
| Mutación con alta probabilidad de éxito (toggle, like, reordenar, marcar hecho) | Optimistic UI: refleja el cambio ya, revierte con aviso (toast con el motivo) si el servidor falla | Optimistic en pagos, borrados o acciones no reversibles |

## Los 4 estados obligatorios de toda vista con datos

Ninguna vista está terminada sin sus 4 estados implementados. Al construir, escribe los 4 desde el inicio:

1. **Cargando**: skeleton con la forma del contenido final.
2. **Vacío**: NUNCA un "no hay datos" seco. Formato: qué significa este vacío + acción sugerida para salir de él ("Aún no tienes citas. [Agendar la primera]"). El primer uso del producto ES el estado vacío: se diseña, no se deja caer.
3. **Error**: qué falló en lenguaje del usuario (no códigos ni jerga) + botón de reintento + los datos del usuario intactos.
4. **Con datos / éxito**: el estado feliz, incluyendo el caso "muchos datos" (¿paginación? ¿scroll?) y el caso "dato extremo" (texto larguísimo, cifra enorme).

## Reglas de animación

1. **Duraciones**: micro-interacciones (hover, pressed, toggle, aparición de tooltip) 150–200ms; transiciones de elementos (modal, drawer, toast, expandir) 200–300ms; nunca >400ms en UI funcional. Más lento = la UI se siente pesada.
2. **Easing**: `ease-out` para elementos que entran (rápido al inicio, aterriza suave), `ease-in` o `ease-in-out` para los que salen. Nunca `linear` para movimiento espacial.
3. **Anima solo `transform` y `opacity`** (composited, 60fps). Prohibido animar propiedades de layout (`width`, `height`, `top`, `margin`) salvo técnica específica que no cause reflow por frame.
4. **`prefers-reduced-motion` se respeta SIEMPRE**: con la preferencia activa, las animaciones de movimiento se reducen a fades o se eliminan. Es requisito de accesibilidad, no opcional.
5. **La animación comunica causa-efecto, nunca decora.** Cada animación debe responder "¿qué le explica esto al usuario?": de dónde vino el elemento (el modal emerge del centro, el drawer del borde), a dónde se fue (el item borrado se colapsa, el toast sale hacia su borde), qué cambió (el toggle se desliza). Si la respuesta es "se ve cool" → se elimina. Animaciones de entrada en cascada en toda la página, parallax decorativo y elementos flotando en loop: fuera, salvo que la dirección visual del design system lo pida explícitamente para una landing.
6. **Consistencia**: mismos patrones de entrada/salida para el mismo tipo de elemento en toda la app. El modal siempre entra igual.

## Checklist de done (feedback) por feature

- [ ] Toda acción tiene acuse <100ms (recorre cada botón/interacción de la feature)
- [ ] Los 4 estados de cada vista implementados
- [ ] Errores conservan el input del usuario y ofrecen salida (retry/corregir)
- [ ] Acciones destructivas confirman nombrando el objeto; las reversibles ofrecen undo
- [ ] Animaciones dentro de rangos de duración, solo transform/opacity, con reduced-motion respetado
