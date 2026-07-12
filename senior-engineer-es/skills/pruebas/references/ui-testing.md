# Pruebas de front y UI/UX

Herramienta por defecto: **Playwright** (o la que el proyecto ya use — detectar con `ls e2e/ tests/ playwright.config.* cypress.config.*` antes de introducir una nueva).

## Regla central

Una prueba de UI que solo verifica que algo "renderiza" no prueba nada. Todo caso de UI **ejerce una interacción y verifica el efecto** — y el efecto que vale es el persistido: tras recargar la página, el dato sigue/no está.

## Reglas de implementación (Playwright)

1. **Selectores estables:** `getByRole` (primero), `getByLabel`, `getByTestId`. Prohibido: clases CSS, XPath posicional, `nth-child`. Si no hay selector estable, repórtalo como deuda — no construyas uno frágil.
2. **Esperas por estado, nunca por tiempo:** `await expect(locator).toBeVisible()` / `toHaveText()`. `waitForTimeout` está prohibido; si el test solo pasa con un sleep, es flaky y se reporta como FALLÓ.
3. **Cada test parte de estado conocido:** login/seed propio, no depende del orden de otros tests.
4. **Verificar efecto, no solo UI optimista:** muchos frontends muestran éxito antes de confirmar el servidor. Tras la acción: recargar y re-verificar, o consultar la API/DB.

## Checklist de UX — los casos que siempre se omiten

Para cada flujo con formularios/mutaciones, el plan incluye estos casos (marcar N/A explícito si no aplica):

| # | Caso | Qué verificar |
|---|---|---|
| 1 | Estado de carga | Al disparar la acción hay feedback visible (spinner/disabled); la UI no queda "muerta". |
| 2 | Estado vacío | Lista/tabla sin datos muestra un empty state con siguiente acción, no una tabla en blanco ni un error. |
| 3 | Error de servidor visible | Con el backend devolviendo 500 (o apagado), el usuario ve un mensaje claro — no pantalla congelada ni error solo en consola. |
| 4 | Doble click en submit | Click rápido x2 → UNA sola entidad creada (verificar contando tras reload). |
| 5 | Formulario inválido | Enviar datos inválidos → mensaje específico JUNTO al campo, el submit no procede, y el server también lo rechaza (probar vía API directa: la validación de UI no es enforcement). |
| 6 | Botón atrás | Tras completar el flujo, back del navegador no duplica la acción ni rompe el estado. |
| 7 | Sesión expirada a media acción | Con token/cookie invalidado, la siguiente acción redirige a login con mensaje — no falla en silencio ni pierde el trabajo sin avisar. |
| 8 | Permisos en UI + API | Lo que el rol no puede hacer no se muestra (UX) Y la API lo rechaza aunque se llame directo (enforcement). |

## Responsive

1. Detectar los breakpoints reales del proyecto (config de Tailwind/design tokens/CSS) — no inventar.
2. Probar los flujos críticos en viewports concretos: al menos **375×667** (móvil), **768×1024** (tablet) y **1280×720** (desktop), ajustados a los breakpoints detectados.
3. Qué verificar por viewport: nada esencial queda oculto o cortado, no hay scroll horizontal, los targets táctiles son usables, la navegación (menú colapsado) funciona.

## Accesibilidad mínima

- Todo el flujo crítico es completable **solo con teclado** (Tab en orden lógico, Enter/Escape funcionan, el foco es visible).
- Inputs con label asociado (si `getByLabel` no lo encuentra, es un bug de accesibilidad — repórtalo).
- Imágenes/íconos con acción tienen nombre accesible (`getByRole('button', { name: ... })` debe poder encontrarlos).
- Contraste: texto esencial legible; si hay duda, screenshot y marcar para revisión.

## Verificación visual

Cuando el cambio es de UI (layout, estilos, componente nuevo):

1. Screenshot del estado final en los 3 viewports (`page.screenshot`), guardados con nombre descriptivo.
2. Compararlos contra lo esperado (diseño/spec) y citarlos como evidencia en el veredicto.
3. Si algo se ve roto pero "los tests pasan", el resultado es FALLÓ — la evidencia visual manda sobre la aserción incompleta.
