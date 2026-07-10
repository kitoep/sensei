# Branches y PRs — alcance, tamaño y operaciones prohibidas

## Branches

**Regla:** branch por unidad de trabajo. Nunca trabajes directo sobre `main`/`master`/`develop` — si ya estás parado en el branch por defecto, crea el branch ANTES del primer commit.

Naming: sigue la convención que ya use el repo (`git branch -r` para verla). Si no hay convención: `<tipo>/<descripcion-corta-kebab>` — `feat/checkout-oxxo`, `fix/descuento-iva`, `chore/actualiza-eslint`. Con tracker: incluye el ID (`feat/YU-482-checkout-oxxo`).

Un branch = un PR futuro. Si mientras trabajas aparece un bug sin relación: NO lo arregles en este branch. Anótalo, o haz branch aparte desde `main` con ese fix.

Antes de abrir el branch, parte de base actualizada: `git fetch origin && git switch -c feat/x origin/main`. Un branch nacido de una base vieja genera conflictos que contaminan el PR.

## Alcance de un PR

**Un PR = un tema que un revisor puede auditar en una sesión.**

| Métrica | Umbral | Al superarlo |
|---|---|---|
| Líneas netas de cambio (sin lockfiles/generados) | ~400 | Evalúa partir en PRs apilados |
| Líneas netas | ~1000 | PROHIBIDO como PR único salvo caso mecánico (abajo) |
| Temas distintos (feature + refactor no relacionado + fix ajeno) | >1 | Partir, siempre |
| Archivos tocados | ~20 | Señal de alerta: ¿es un tema o varios? |

Excepción mecánica: un cambio masivo pero uniforme (rename global, formateo automático, regenerar lockfile, codemod) puede ser un PR grande — SIEMPRE solo, jamás mezclado con cambios de lógica, y declarando en la descripción el comando que lo generó para que el revisor lo reproduzca en vez de leerlo.

### Cómo partir trabajo grande: PRs apilados

Parte por capas entregables, cada una en verde y revisable por sí sola:

1. **PR 1 — preparación:** refactors/renombres que habilitan el feature, sin cambio de comportamiento. Base: `main`.
2. **PR 2 — núcleo:** el cambio de comportamiento mínimo (modelo + lógica + tests). Base: branch del PR 1.
3. **PR 3 — superficie:** UI, endpoints adicionales, migración de consumidores. Base: branch del PR 2.

Reglas del stack:

- Cada PR declara en su descripción de cuál depende ("Apilado sobre #12; revisar solo el último commit range").
- Se mergean en orden; tras mergear PR 1, re-basa el PR 2 sobre `main` (esto SÍ es rebase permitido: es tu branch de PR, avisando en el PR).
- Alternativa cuando el stack no vale la pena: mergear PR 1 primero y abrir PR 2 después, en serie.
- Feature incompleto pero seguro (dark launch / feature flag apagado) es la forma estándar de mergear núcleo sin exponer superficie a medias.

## Operaciones prohibidas sin pedido explícito del usuario

"Explícito" = el usuario lo pidió con esas palabras en ESTA conversación. "Lo hice antes en otro PR" no es permiso.

| Operación | Por qué | Alternativa permitida |
|---|---|---|
| Commit/push directo a `main`/`master`/`develop` | Salta revisión y CI de PR | Branch + PR |
| `git push --force` / `--force-with-lease` a branch compartido o `main` | Borra trabajo ajeno | Commit nuevo; si es TU branch de PR y necesitas re-basar el stack, `--force-with-lease` solo sobre tu branch avisando en el PR |
| `git commit --amend` de un commit ya pusheado | Reescribe historia que otros pueden tener | Commit nuevo ("fixup: ...") |
| `git rebase` de commits ya pusheados en branch compartido | Ídem | Merge de `main` hacia tu branch para actualizarte |
| `git push --delete` / borrar branches remotos ajenos | Destructivo e irrecuperable | Pedir confirmación con el nombre exacto |
| `--no-verify`, desactivar hooks o firmas | Los hooks existen por algo | Arreglar lo que el hook detecta |
| `git reset --hard` / `git checkout --` sobre cambios que no creaste tú | Puede destruir trabajo del usuario | `git stash` (recuperable) + avisar |

Antes de cualquier operación de esta tabla que el usuario SÍ pidió: di qué vas a ejecutar y qué efecto tiene, en una línea, y ejecútala tal cual.

## Antes de abrir el PR — checklist

- [ ] Branch actualizado respecto a `main` (merge o rebase local pre-push) y sin conflictos.
- [ ] La suite pasa en el último commit (`X passed, 0 failed` — pega el resumen en la descripción).
- [ ] El diff del PR (`git diff main...HEAD`) contiene SOLO el tema del PR. Cambio colado → sácalo a otro branch.
- [ ] Sin archivos prohibidos (checklist de `commits.md`) en TODO el rango, no solo el último commit: `git diff main...HEAD --stat`.
- [ ] Descripción completa según `descripcion-pr.md`.
- [ ] Título del PR = primera línea de commit bien escrita: imperativo, específico, con prefijo/ticket si el repo lo usa.
