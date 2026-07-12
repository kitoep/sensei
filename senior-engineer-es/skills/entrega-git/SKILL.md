---
name: entrega-git
description: Use when about to run git add, git commit, or git push; when creating a branch or pull request; when writing a commit message or PR description; when the user says "haz commit", "sube los cambios", "crea el PR", or "mergea"; when deciding what goes in a PR or how to split large work; or before any history-rewriting operation (amend, rebase, force-push).
---

# Entrega-git — commits, branches y PRs auditables

## Principio

El trabajo no se entrega cuando el código funciona: se entrega cuando un revisor humano puede auditarlo. La unidad de entrega es el commit atómico y el PR de un solo tema con descripción verificable. Un volcado de 2000 líneas con mensaje "updates" no es una entrega; es una deuda que pagará el revisor.

**Violar la letra de estas reglas es violar su espíritu.** No hay cambios "demasiado pequeños" para el proceso ni prisa que justifique `git add .` a ciegas.

## Protocolo (carga el reference ANTES de actuar)

| Momento | Qué haces | PUERTA para avanzar | Reference |
|---|---|---|---|
| Antes de `git add` | Revisas el diff completo archivo por archivo | Puedes nombrar la intención de CADA archivo staged; cero secretos/builds/temporales | `references/commits.md` |
| Antes de `git commit` | Una intención por commit + mensaje con el porqué | El mensaje responde "¿por qué?" y el diff staged contiene solo esa intención | `references/commits.md` |
| Antes de crear branch/PR | Branch por unidad de trabajo; alcance de un tema revisable | El PR cabe en una revisión (<~400 líneas netas) o está partido en PRs apilados | `references/branches-prs.md` |
| Antes de escribir la descripción | Plantilla completa de descripción auditable | Las 5 secciones llenas, con evidencia de verificación pegada | `references/descripcion-pr.md` |
| Antes de push/amend/rebase/force | Verificas que la operación esté permitida | El usuario la pidió explícitamente, o es push normal de tu branch | `references/branches-prs.md` |

## Prohibido sin pedido explícito del usuario

- Commitear o pushear directo a `main`/`master`/`develop`.
- `push --force` (en cualquier variante) a un branch compartido.
- `commit --amend` o `rebase` de commits ya pusheados.
- Saltarse hooks (`--no-verify`) o firmas.

## Al terminar el trabajo: el destino lo decide el usuario

Cuando el trabajo del branch quedó completo, no decidas tú qué sigue. Primero corre la suite de tests (si falla algo, no hay cierre: se arregla primero) y luego presenta este menú tal cual:

1. Mergear a la base localmente
2. Pushear y abrir un PR
3. Dejar el branch como está (el usuario lo maneja después)
4. Descartar este trabajo

Ejecutas la opción que elija, y nada más.

## Red flags — DETENTE si te descubres pensando esto

- "`git add .` y listo, ya sé qué cambié" → no lo sabes. Archivos accidentales, `.env` y builds entran exactamente así. Diff primero.
- "Lo meto todo en un commit, total es el mismo feature" → si mezcla refactor + feature + formato, son 3 commits. Sepáralos.
- "El mensaje da igual, el diff se explica solo" → el diff dice QUÉ cambió; solo el mensaje puede decir POR QUÉ. Escríbelo.
- "La descripción del PR la genero resumiendo el diff" → re-narrar el diff es ruido. El revisor ya ve el diff; necesita contexto, verificación y riesgos.
- "Hago force-push para dejar la historia limpia" → historia compartida no se reescribe sin pedido explícito.
- "Es un cambio chico, directo a main" → los incidentes de producción también son chicos. Branch + PR.

## Racionalizaciones conocidas

| Excusa | Realidad |
|---|---|
| "Revisar el diff completo toma mucho tiempo" | Toma 2 minutos. Un secreto commiteado obliga a rotar credenciales y purgar historia: horas, más el incidente. |
| "Nadie lee los mensajes de commit" | Se leen exactamente cuando algo se rompe (`git log`, `git blame`, bisect). Ahí "fix stuff" cuesta horas de arqueología. |
| "Un PR grande es más eficiente que tres chicos" | Un PR de 2000 líneas recibe aprobación sin lectura real o semanas de ping-pong. Tres PRs apilados se revisan en serio y se mergean antes. |
| "El PR ya describe qué hice, corre los tests el CI" | El CI verifica que compila y pasa la suite; no verifica que el cambio hace lo que el PR promete. La evidencia de verificación va en la descripción. |
| "Amendeo el commit para no ensuciar la historia" | Si ya está pusheado, amendear reescribe historia que otros pueden tener. Commit nuevo. |
