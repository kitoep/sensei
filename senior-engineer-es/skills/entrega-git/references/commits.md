# Commits — atómicos, revisados, con el porqué

## Regla central: una intención por commit

Un commit atómico responde a UNA pregunta: "¿qué intención cumple este cambio?". Si la respuesta necesita la palabra "y" ("agrega el endpoint **y** refactoriza el cliente **y** arregla el formato"), son varios commits.

Clasifica cada cambio de tu working tree en una de estas intenciones y commitea por separado:

| Intención | Ejemplo |
|---|---|
| Feature | Nuevo endpoint, nueva pantalla, nueva regla de negocio |
| Fix | Corrección de comportamiento defectuoso |
| Refactor | Misma conducta, mejor estructura |
| Formato/estilo | Prettier, imports, whitespace — cero cambio semántico |
| Infra/config | Dependencias, CI, dotfiles del proyecto |
| Docs/tests solos | Cambios que no tocan código de producción |

**Prohibido mezclar refactor con feature/fix en el mismo commit.** El revisor no puede distinguir "movió código" de "cambió comportamiento" en un diff mezclado — que es exactamente donde se esconden los bugs.

## PUERTA antes de `git add` — el diff completo, archivo por archivo

`git add .` / `git add -A` a ciegas está PROHIBIDO. Secuencia obligatoria:

1. `git status` — lista completa de modificados y untracked.
2. `git diff` (y `git diff --stat`) — lees el diff de CADA archivo modificado.
3. Por cada archivo untracked: ábrelo o explica por qué existe.
4. Staging selectivo: `git add <ruta> <ruta>` (o `git add -p` si un archivo mezcla intenciones).
5. `git diff --staged` — verificación final: lo staged es exactamente UNA intención.

**La puerta:** puedes nombrar la intención de cada archivo staged. Si hay un archivo que no sabes por qué cambió, NO se commitea — se investiga.

### Checklist de archivos prohibidos (revisa en cada commit)

- [ ] Secretos: `.env`, `.env.*`, credenciales, tokens, keys, certificados, `*.pem`. Si un diff agrega un valor que parece secreto (aunque sea en código), detente y pregunta.
- [ ] Artefactos de build: `dist/`, `build/`, `node_modules/`, `__pycache__/`, `*.pyc`, binarios compilados.
- [ ] Temporales y locales: `*.log`, dumps, `tmp/`, archivos de scratch, notas personales, salidas de debug.
- [ ] Config personal de IDE: `.vscode/settings.json`, `.idea/` (salvo que el repo ya los versione a propósito).
- [ ] Cambios accidentales: `console.log`/`print` de debug, comentarios TODO tuyos de la sesión, código comentado "por si acaso".

Si algo de esto ya está trackeado por error, ese es un commit propio ("remove committed build artifacts") + entrada en `.gitignore` — no lo escondas dentro de otro commit.

**Secreto commiteado (aunque no pusheado aún):** no basta borrarlo en un commit siguiente si ya salió del repo local. Si se pusheó: la credencial se considera comprometida — avisa al usuario, rota la credencial, y luego decide si purgar historia. Nunca lo resuelvas en silencio.

## Formato del mensaje

```
<verbo imperativo> <qué, en ≤72 caracteres>

<POR QUÉ: el problema o la necesidad que motiva el cambio.
Qué alternativa se descartó y por qué, si la hubo.
Efectos no obvios o riesgos, si los hay.>
```

Reglas:

- **Primera línea:** imperativo ("agrega", "corrige", "elimina" / "add", "fix", "remove" — usa el idioma que ya use el `git log` del repo), específica, ≤72 caracteres. Si el repo usa convención (Conventional Commits, prefijos de ticket), síguela: `git log --oneline -20` antes de tu primer commit.
- **El cuerpo lleva el PORQUÉ, no el cómo.** El diff ya muestra el cómo. Prohibido el cuerpo que re-narra el diff ("cambia X por Y en el archivo Z"). Test: si borro el cuerpo y leo el diff, ¿perdí información? Si no, el cuerpo no dice nada.
- Cuerpo obligatorio cuando: el cambio corrige un bug (referencia el síntoma), toma una decisión no obvia, o tiene efectos colaterales. Un cambio trivial y autoexplicativo puede ir sin cuerpo.
- Referencia al ticket/issue si existe.

| Mensaje | Veredicto |
|---|---|
| `updates` / `fix stuff` / `wip` / `cambios` | Prohibido. No dice nada. |
| `fix: corrige el cálculo de descuento` | Insuficiente si el cuerpo no dice cuál era el bug. |
| `fix: usa precio con IVA en el cálculo de descuento`<br><br>`El descuento se calculaba sobre el precio sin IVA, cobrando de más en órdenes con cupón (reporte #482). El precio con IVA ya está disponible en el line item; no se recalcula.` | Correcto: qué + por qué + referencia. |

## Cuándo (y cómo) separar en varios commits

Señales de que tu working tree son varios commits:

- Tocaste archivos para "aprovechar el viaje" (formateo, renombres) además del cambio pedido.
- Hiciste un refactor preparatorio antes del feature/fix.
- Hay cambios en áreas sin relación (backend + script de deploy + docs de otra cosa).

Orden recomendado: primero los commits preparatorios (refactor, formato) — cada uno debe dejar la suite en verde — y al final el feature/fix, que queda como un diff mínimo y legible. Usa `git add -p` para partir archivos que mezclan intenciones.

**Cada commit debe compilar y pasar la suite por sí solo.** Un commit intermedio roto rompe `git bisect` y anula el valor de haber separado.
